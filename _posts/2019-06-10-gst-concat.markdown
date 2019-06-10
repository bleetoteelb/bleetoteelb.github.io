---
layout: post
title:  "Concat plugin 분석"
subtitle:   "Concat plugin 뷴석"
categories: devlog
tags: gst
---

[TOP](README.md)

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>


# Introduction
 본 문서에서는 concat 엘리먼트의 테스트코드 및 예제 프로그램을 사용하여 동작성을 확인해 볼 수 있도록 한다. concat 엘리먼트의 실제 코드도 분석하며 추후에 Playbin 구동시에 이해를 돕고자한다.

## 1) Reference
* * *
* [concat](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer-plugins/html/gstreamer-plugins-concat.html)
* [gstconcat.h](https://gitlab.freedesktop.org/gstreamer/gstreamer/blob/master/plugins/elements/gstconcat.h)
* [gstconcat.c](https://gitlab.freedesktop.org/gstreamer/gstreamer/blob/master/plugins/elements/gstconcat.c)
* [Unit Test Code](https://gitlab.freedesktop.org/gstreamer/gstreamer/blob/master/tests/check/elements/concat.c)

  
## 2) Concat 코드 분석
* * *
__GOAL__ : Unit Test Code 를 실행 및 분석하면서 gstconcat.c(.h) 파일의 코드를 같이 분석하여 Pad(capability) 및 생성, event/chain/query function, change state function 등에 학습할 수 있도록 한다.


### 2-1) Concat plugin 소개

&nbsp; Concat plugin은 스트림들을 하나의 연속 스트림으로 연결하는 plugin이다. concat element에 들어오는 stream 중, 처리중인 stream을 제외한 다른 모든 stream들은 이전 pad의 처리가 끝날 때까지(EOS를 받을 때까지) 기다린다. GST_FORMAT_TIME segment와 GST_FORMAT_BYTES segment들의 연속성을 유지하도록 배치한다.

```c

 ...

GObject
 +----GInitiallyUnowned
       +----GstObject
             +----GstElement
                   +----GstConcat

Pad Templates:
  SINK template: 'sink_%u'
    Availability: On request
    Capabilities:
      ANY
  
  SRC template: 'src'
    Availability: Always
    Capabilities:
      ANY

Element has no clocking capabilities.
Element has no URI handling capabilities.

Pads:
  SRC: 'src'
    Pad Template: 'src'

Element Properties:
  active-pad          : Currently active src pad
                        flags: readable
                        Object of type "GstPad"
  adjust-base         : Adjust the base value of segments to ensure they are adjacent
                        flags: readable, writable
                        Boolean. Default: true
  name                : The name of the object
                        flags: readable, writable, 0x2000
                        String. Default: "concat0"
  parent              : The parent of the object
                        flags: readable, writable, 0x2000
                        Object of type "GstObject"
```

### &nbsp;&nbsp;&nbsp;&nbsp; __설명__
- _GObject_ - _GstElement_ - _GstConcat_ 순으로 상속되는 element이다.
- _source pad_ 는 element를 생성할때 자동으로 생성하며, 오로지 한개만 존재한다.
- _sink pad_ 는 (요청될 때) 여러개 만들어질 수 있다.

&nbsp;  
- __(GstPad) active-pad__ : _sink pad_ 가 여러개이기 때문에, 각 thread에서 작업 중인 pad를 명시해 주어야 한다.
- __(Gboolean) adjust-base__ : segments들이 인접해 있는지를 확인하는 속성값이다. 기본값은 TRUE.
- __(gchar) name__ : object의 이름을 저장하는데, 따로 지정이 안되어있으면 "_concat0_"이다.
- __(GstObject) parent__ : 상위 object를 저장한다. "TestPipeline" 이라는 파이프라인에 concat element를 등록했다면 parent는 TestPipeline이 된다.

&nbsp; 


<center><img src="https://bleetoteelb.github.io/assets/img/dotgraph_concat1.png" width="700"></center>
<center><img src="https://bleetoteelb.github.io/assets/img/dotgraph_concat2.png" width="700"></center>



### 2-2) 함수 소개

함수들...
- gst_concat_dispose ( ... )
- gst_concat_finalize ( ... )
- gst_concat_get_property ( ... )
- gst_concat_set_property ( ... )
- gst_concat_change_state ( ... )
- gst_concat_request_new_pad ( ... )
- gst_concat_release_pad ( ... )
- gst_concat_sink_chain ( ... )
- gst_concat_sink_event ( ... )
- gst_concat_sink_query ( ... )
- gst_concat_src_event ( ... )
- gst_concat_src_query ( ... )
- gst_concat_switch_pad ( ... )
- gst_concat_notify_active_pad ( ... )
- gst_concat_class_init ( ... )
- gst_concat_pad_wait ( ... )
- reset_pad ( ... )
- unblock_pad ( ... )

<br>

#### 2-2-1) gst_concat_change_state  
&nbsp; Param : <span class="fill_color">GstElement * element</span>, <span class="fill_color">GstStateChange transition</span>

&nbsp; 인자로 받은 transition의 요청대로 상태를 변경한다. 이 상태변화는 parent_element에도 전파된다.

- __GST_STATE_GHANGE_READY_TO_PAUSED__ : 각 pad에 reset_pad를 실행한다. (reset_pad : flushing을 FALSE로 바꾸고, segment_init을 실행)
- __GST_STATE_CHANGE_PAUSED_TO_READY__ : 각 pad에 unblock_pad를 실행한다. (unblock_pad : flushing을 TRUE로 바꾼다.)

<br>

#### 2-2-2) gst_concat_request_new_pad

&nbsp; Param : <span class="fill_color">GstElement * element</span>, <span class="fill_color">GstPadTemplate * templ</span>, <span class="fill_color">const gchar * name</span>, <span class="fill_color">const GstCaps * caps</span>

&nbsp; name으로 받은 문자열로 sink pad를 만들어서 element에 추가한 후 pad를 반환한다. 이 때, sink와 관련된 chain, event, query 함수를 sink에 연결하고, pad를 활성화 시킨다. 

&nbsp; 함수 이름은 new_pad 이지만, concat element는 src가 기본생성, sink가 요청 시 생성이므로, sink pad만 생성 한다.

<br>

#### 2-2-3) gst_concat_release_pad
&nbsp; Param : <span class="fill_color">GstElement * element</span>, <span class="fill_color">GstPad * pad</span>

&nbsp; 해당 pad에서 할 일이 끝났을 경우, pad를 element로부터 제거하는 역할을 한다. flushing을 TRUE로 바꾸고 (원래는 flush가 될 때 true), pad를 비활성화시키며, 만약 다음 pad가 없고 EOS 상태이면 EOS event를 push하는 동작까지 담당한다.  

<br>

#### 2-2-4) gst_concat_sink_chain
&nbsp; Param : <span class="fill_color">GstPad * pad</span>, <span class="fill_color">GstObject * parent</span>, <span class="fill_color">GstBuffer * buffer</span>  

&nbsp; sink에서의 처리가 끝나면, sink의 parent element (여기서는 concat)의 source pad에 연결된 chain pad로 buffer를 전달한다. 

<br>

&nbsp; 
#### 2-2-5) gst_concat_switch_pad 
&nbsp; Param : <span class="fill_color">GstConcat * self</span>  

&nbsp; pad간의 sync를 맞춘 뒤 (stop, start 등의 위치를 이용하여) element내 다음 sink pad가 있는지를 반환한다. 다음 sinkpad가 있으면 1 , 없으면 0을 반환한다.
```c
  next = self->current_sinkpad != NULL;

  self->last_stop = GST_CLOCK_TIME_NONE;

  return next;
```

<br>

#### 2-2-6) gst_concat_sink_event 
&nbsp; Param : <span class="fill_color">GstPad * pad</span>, <span class="fill_color">GstObject * parent</span>, <span class="fill_color">GstEvent * event</span>

&nbsp; 인자로 받은 event의 type에 따라서 각각 다른 event를 처리하고 결과를 boolean으로 반환한다.

- __GST_EVENT_STREAM_START__ : Stream이 시작되어 event_default를 발생시킨다.
- __GST_EVENT_SEGMENT__ : Segment의 형식에 대한 확인한 후, 새로운 Segment라는 event를 발생시킨다(<span class="fill_color">gst_pad_push_event</span> 이용).
- __GST_EVENT_EOS__ : switch_pad를 호출하여 pad를 교체하고, Stream의 끝을 알리는 event를 발생시킨다(<span class="fill_color">gst_pad_push_event</span> 이용).
- __GST_EVENT_FLUSH_START__ : pad의 flushing을 TRUE로 바꾸고, event_default를 발생시켜 내부적으로 pad와 연결된 모든 작업을 중지시킵니다.
- __GST_EVNET_FLUSH_STOP__ : pad의 flushing을 FALSE로 바꾸고, event_default를 발생시켜 내부적으로 pad와 연결된 모든 작업을 중지시킵니다.

<br>

#### 2-2-7) gst_concat_sink_query 
&nbsp; Param : <span class="fill_color">GstPad * pad</span>, <span class="fill_color">GstObject * parent</span>, <span class="fill_color">GstQuery * query</span>

&nbsp; query를 받았을 경우, flushing상황이면 FALSE를 반환, 아니라면 이전 작업을 기다린 후 query_default를 호출한다.

<br>

#### 2-2-8) gst_concat_src_event 
Param : <span class="fill_color">GstPad * pad</span>, <span class="fill_color">GstObject * parent</span>, <span class="fill_color">GstEvent * event</span>

&nbsp; 인자로 받은 event의 type에 따라서 각각 다른 event를 처리하고 결과를 boolean으로 반환한다.

- __GST_EVENT_SEEK__ : sink pad가 현재 처리중이라면 push_event를 호출한다.
- __GST_EVENT_QOS__ : sink pad가 현재 처리중이라면 QOS(Quality of Service)에 관련된 event를 발생한다. OVERFLOW, UNDERFLOW, THROTTLE 등.
- __GST_EVENT_FLUSH_STOP__ : event_default를 호출한다.

<br>

#### 2-2-9) gst_concat_src_query 
&nbsp; Param : <span class="fill_color">GstPad * pad</span>, <span class="fill_color">GstObject * parent</span>, <span class="fill_color">GstQuery * query</span>

&nbsp; query를 받았을 경우, query_default를 호출한다.

<br>


#### 2-2-10) gst_concat_pad_wait
&nbsp; Param : <span class="fill_color">GstConcatPad * spad</span>, <span class="fill_color">GstConcat * self</span>

&nbsp; 함수를 호출한 Thread의 Pad가 현재 처리중인 pad인지를 확인한다. 다른 Pad의 작업이 끝나고 해당 pad의 차례가 될 때까지 기다리다가 TRUE를 반환한다.  Pad가 flushing 상태이면, 별도의 과정없이 바로 FALSE를 반환한다. 

```c
  while (spad != GST_CONCAT_PAD_CAST (self->current_sinkpad)) {
    GST_TRACE_OBJECT (spad, "Not the current sinkpad - waiting");
    if (self->current_sinkpad == NULL && g_list_length (self->sinkpads) == 1) {
      GST_LOG_OBJECT (spad, "Sole pad waiting, switching");
      /* If we are the only sinkpad, take active pad ownership */
      self->current_sinkpad = gst_object_ref (self->sinkpads->data);
      break;
    }
    g_cond_wait (&self->cond, &self->lock);
    if (spad->flushing) {
      g_mutex_unlock (&self->lock);
      GST_DEBUG_OBJECT (spad, "Flushing");
      return FALSE;
    }
  }
```



<br>


## 3) Unit Test Code
* * *
__GOAL__ :  concat 엘리먼트의 유닛 테스트 코드를 실행시켜 보고 (e.g make elements/concat.check) concat의 동작성을 확인 및 분석해 본다.

&nbsp; Concat에 대한 Test는 TIMESTAMP와 BYTES 두 경우에 대해 시행되었지만, 속성의 종류 외에 다른 동작들은 유사하므로 TIMESTAMP에 대해서만 설명한다.


__TEST Code 전문__
```c
GST_START_TEST (test_concat_simple_time)
{
  GstElement *concat;
  GstPad *sink1, *sink2, *sink3, *src, *output_sink;
  GThread *thread1, *thread2, *thread3;

  got_eos = FALSE;
  buffer_count = 0;
  gst_segment_init (&current_segment, GST_FORMAT_UNDEFINED);
  
  concat = gst_element_factory_make ("concat", NULL);
  fail_unless (concat != NULL);

  sink1 = gst_element_get_request_pad (concat, "sink_%u");
  fail_unless (sink1 != NULL);

  sink2 = gst_element_get_request_pad (concat, "sink_%u");
  fail_unless (sink2 != NULL);

  sink3 = gst_element_get_request_pad (concat, "sink_%u");
  fail_unless (sink3 != NULL);

  src = gst_element_get_static_pad (concat, "src");

  output_sink = gst_pad_new ("sink", GST_PAD_SINK);
  fail_unless (output_sink != NULL);
  fail_unless (gst_pad_link (src, output_sink) == GST_PAD_LINK_OK);

  gst_pad_set_chain_function (output_sink, output_chain_time);
  gst_pad_set_event_function (output_sink, output_event_time);

  gst_pad_set_active (output_sink, TRUE);

  fail_unless (gst_element_set_state (concat,
          GST_STATE_PLAYING) == GST_STATE_CHANGE_SUCCESS);

  thread1 = g_thread_new ("thread1", (GThreadFunc) push_buffers_time, sink1);
  thread2 = g_thread_new ("thread2", (GThreadFunc) push_buffers_time, sink2);
  thread3 = g_thread_new ("thread3", (GThreadFunc) push_buffers_time, sink3);

  g_thread_join (thread1);
  g_thread_join (thread2);
  g_thread_join (thread3);

  fail_unless (got_eos);
  fail_unless_equals_int (buffer_count, 3 * N_BUFFERS);

  gst_element_set_state (concat, GST_STATE_NULL);
  gst_pad_unlink (src, output_sink);
  gst_object_unref (src);
  gst_element_release_request_pad (concat, sink1);
  gst_object_unref (sink1);
  gst_element_release_request_pad (concat, sink2);
  gst_object_unref (sink2);
  gst_element_release_request_pad (concat, sink3);
  gst_object_unref (sink3);
  gst_pad_set_active (output_sink, FALSE);
  gst_object_unref (output_sink);
  gst_object_unref (concat);

}

GST_END_TEST;
```
&nbsp; 이 부분에서는 다음과 같이 단계를 나눠서 설명한다.
- Concat element 생성
- Fake source pad 생성
- 각 Sink pad 동작 Thread

<br>

### 3-1) Concat element 생성

```c
  concat = gst_element_factory_make ("concat", NULL);
  fail_unless (concat != NULL);
```


<div style="width:800px;height:220px;">
  <div style="width:300px;float:left;margin-right:30px;margin-left:30px">
    <img src="https://bleetoteelb.github.io/assets/img/concat_element1.png">
  </div>
    
  <div style="width:440px;float:right;"> 
    <br>
    <div>
      &nbsp; concat이라는 이름을 가진 factory를 만든다. 
      이 때, gstreamer 안에 concat이라는 플러그인이 이미 등록되어 있어야하고 
      srpad는 concat element가 만들어질 때 자동으로 생성된다.</span>
    </div>
    <br>
    <div>
      &nbsp; concat이 제대로 생성되었는지 fail_unless를 이용해 확인한다.
    </div>
    </span>
  </div>
</div>

<div style="clear:both:"></div>



```c
  sink1 = gst_element_get_request_pad (concat, "sink_%u");
  fail_unless (sink1 != NULL);

  sink2 = gst_element_get_request_pad (concat, "sink_%u");
  fail_unless (sink2 != NULL);

  sink3 = gst_element_get_request_pad (concat, "sink_%u");
  fail_unless (sink3 != NULL);
```



<div style="width:800px;height:220px;">
  <div style="width:300px;float:left;margin-right:30px;margin-left:30px">
    <img src="https://bleetoteelb.github.io/assets/img/concat_element2.png">
  </div>
    
  <div style="width:440px;float:right;"> 
  <br>
    <div>
      &nbsp; sink0,sink1,sink2를 만들어서 concat element에 추가한다. 
      Test code의 <span class="fill_color">gst_element_get_request_pad</span>는 concat plugin의 <span class="fill_color">gst_concat_request_new_pad</span> 함수를 호출한다.
    </div>
    <br>
    <div>
      &nbsp; fail_unless함수를 이용해 각 sink가 제대로 생성되었는지를 확인한다.
    </div>
  </div>
</div>

<br>

<br>

### 3-2) Fake source pad생성

```c
  output_sink = gst_pad_new ("sink", GST_PAD_SINK);
  fail_unless (output_sink != NULL);
```

<div style="width:800px;height:220px;">
  <div style="width:400px;float:left;margin-right:30px;margin-left:30px">
    <img src="https://bleetoteelb.github.io/assets/img/concat_element3.png">
  </div>
    
  <div style="width:340px;float:right;"> 
    <br>
    <div>
    &nbsp; <span class="fill_color">gst_pad_new</span>함수를 이용해
    srcpad에서 전달한 data를 받을 sink pad를 생성한다. 
    이 sink pad에 대한 chain 함수와 event함수는 Test code안에 별도로 구현되어있다.
    </div>
    <br>
    <div>
      &nbsp; fail_unless함수를 이용해 각 output_sink가 제대로 생성되었는지를 확인한다.
    </div>
  </div>
</div>



```c
fail_unless (gst_pad_link (src, output_sink) == GST_PAD_LINK_OK);
```
<div style="width:800px;height:220px;">
  <div style="width:400px;float:left;margin-right:30px;margin-left:30px">
    <img src="https://bleetoteelb.github.io/assets/img/concat_element4.png">
  </div>
    
  <div style="width:340px;float:right;"> 
    <br>
    <div>
      &nbsp; <span class="fill_color">gst_pad_link</span> 함수를 이용해
      srcpad와 새로 만든 sink pad 사이의 link를 생성한다. 
    </div>
    <br>
    <div>
      &nbsp; fail_unless함수를 이용해 각 link가 제대로 생성되어 GST_PAD_LINK_OK가 반환되었는지 확인한다.
    </div>
  </div>
</div>

<br>

### 3-3) 각 Sink pad별 Thread 생성

```c
  thread1 = g_thread_new ("thread1", (GThreadFunc) push_buffers_time, sink1);
  thread2 = g_thread_new ("thread2", (GThreadFunc) push_buffers_time, sink2);
  thread3 = g_thread_new ("thread3", (GThreadFunc) push_buffers_time, sink3);
```


<div style="width:800px;height:300px;">
  <div style="width:500px;float:left;margin-right:30px;margin-left:30px">
    <img src="https://bleetoteelb.github.io/assets/img/concat_element5.png">
  </div>
    
  <div style="width:240px;float:right;"> 
    <br>
    <div>
      &nbsp; <span class="fill_color">g_thread_new</span> 함수를 이용해 
      각 sink에 대해 <span class="fill_color">push_buffers_time</span> 함수를 처리하는 thread들을 생성한다.
    </div>
    <br>
    <div>
      &nbsp; fail_unless함수를 이용해 각 link가 제대로 생성되어 GST_PAD_LINK_OK가 반환되었는지 확인한다.
    </div>
  </div>
</div>

 <br>

 <br>

### 3-3) sink로 buffer 전달

```c
  g_thread_join (thread1);
  g_thread_join (thread2);
  g_thread_join (thread3);
```

 &nbsp; <span class="fill_color">g_thread_join</span>함수는 각 thread가 끝날 때까지 callback을 기다린다.
thread1이 작업을 시작하면 concat의 current_pad가 sink_0가 되어 thread2와 thread3는 current_pad의 값이 null이 될 때까지 진행을 멈춘다.
이 과정은 <span class="fill_color">gst_concat_pad_wait</span>함수를 통해 이뤄진다.

&nbsp; 각 thread가 시작되면 <span class="fill_color">push_buffers_time</span> 함수가 실행된다.  
&nbsp; 다음은 <span class="fill_color">push_buffers_time</span> 함수 전문이다.

```c
static gpointer
push_buffers_time (gpointer data)
{
  GstSegment segment;
  GstPad *pad = data;
  gint i;
  GstClockTime timestamp = 0;

  gst_pad_send_event (pad, gst_event_new_stream_start ("test"));
  gst_segment_init (&segment, GST_FORMAT_TIME);
  gst_pad_send_event (pad, gst_event_new_segment (&segment));

  for (i = 0; i < N_BUFFERS; i++) {
    GstBuffer *buf = gst_buffer_new_and_alloc (1000);

    gst_buffer_memset (buf, 0, i, 1);
	
    GST_BUFFER_TIMESTAMP (buf) = timestamp;
    timestamp += 25 * GST_MSECOND;
    GST_BUFFER_DURATION (buf) = timestamp - GST_BUFFER_TIMESTAMP (buf);

    fail_unless (gst_pad_chain (pad, buf) == GST_FLOW_OK);
  }

  gst_pad_send_event (pad, gst_event_new_eos ());

  return NULL;
}
```
&nbsp; 위 코드는 다음 3단계로 나눠진다.

- Segment 시작 event
- Buffer 전달
- EOS 전달

<br>

#### 3-3-1) Segment 시작 event
```c
  gst_pad_send_event (pad, gst_event_new_stream_start ("test"));
  gst_segment_init (&segment, GST_FORMAT_TIME);
  gst_pad_send_event (pad, gst_event_new_segment (&segment));
```

&nbsp; <span class="fill_color">gst_event_new_stream_start</span> 함수는 "test"라는 event를 새로 만든다. 
(함수 내부에서 <span class="fill_color"> gst_event_init</span> 함수를 호출하고, 만들어진 event를 반환한다.)
이후 <span class="fill_color">gst_pad_send_event</span>함수를 호출하는데, 이 때 pad의 parent는 concat이기 때문에, concat의 event함수(<span class="fill_color">gst_concat_sink_event</span>)를 호출한다.

&nbsp; Test code에서는 임의의 stream을 만들것이기 때문에 <span class="fill_color">gst_segment_init</span> 함수를 이용하여 segment를 초기화한다.

_segment_init 함수_
```c
void
gst_segment_init (GstSegment * segment, GstFormat format)
{
  g_return_if_fail (segment != NULL);

  segment->flags = GST_SEGMENT_FLAG_NONE;
  segment->rate = 1.0;
  segment->applied_rate = 1.0;
  segment->format = format;
  segment->base = 0;
  segment->offset = 0;
  segment->start = 0;
  segment->stop = -1;
  segment->time = 0;
  segment->position = 0;
  segment->duration = -1;
}
```

다시 이렇게 segment가 새로 만들어졌다는 event를 다시 보낸다.

<br>

#### 3-3-2) Buffer 전달

```c
#define N_BUFFERS 10

  GstClockTime timestamp = 0;

  for (i = 0; i < N_BUFFERS; i++) {
    GstBuffer *buf = gst_buffer_new_and_alloc (1000);

    gst_buffer_memset (buf, 0, i, 1);
	
    GST_BUFFER_TIMESTAMP (buf) = timestamp;
    timestamp += 25 * GST_MSECOND;
    GST_BUFFER_DURATION (buf) = timestamp - GST_BUFFER_TIMESTAMP (buf);

    fail_unless (gst_pad_chain (pad, buf) == GST_FLOW_OK);
  }
```
&nbsp; <span class="fill_color">gst_buffer_new_and_alloc</span> 함수를 이용해 크기 1000짜리 버퍼를 할당합니다.  
&nbsp; <span class="fill_color">gst_buffer_memset</span> 함수를 통해 _i_ 값으로 1만큼 채워줍니다.
&nbsp; buf의 TIMESTAMP,DURATION 속성에 timestamp, duration을 넣고 <span class="fill_color">gst_pad_chain</span>함수를 호출한다. 여기서 함수 인자인 pad가 concat의 pad이므로 마찬가지로 concat의 <span class="fill_color">gst_concat_sink_chain<span>함수로 연결된다.

![concat_element6](https://bleetoteelb.github.io/assets/img/concat_element6.png)

&nbsp; sink_0에 stream(buffer)가 들어가면(gst_pad_chain), 내부에서 sink_0인 concat으로 올라가서 concat의 srcpad에서 link된 sink pad로 sink_chain함수를 이용해서 buffer를 전달한다(gst_pad_push). gst_pad_push는 다시 gst_pad_push_data를 호출하고 gst_pad_push_data에서 pad의 peer pad(여기서는 output_sink pad)를 가져와서 peer pad의 chain 함수를 호출한다.
이 때, output_sink pad의 chain함수는 처음에 등록한 output_chain_time 함수이다.

정리하면,  
1. <span class="fill_color">push_buffers_time</span> 속 for문
2. <span class="fill_color">gst_pad_chain</span> 으로 buf 보내기
3. 함수 내부에서 pad의 parent인 concat을 가져옴
4. concat의 <span class="fill_color">gst_concat_sink_chain</span> 을 호출
5. <span class="fill_color">gst_pad_push</span> 로 srcpad와 buffer 전달
6. <span class="fill_color">gst_pad_push_data</span> 로 srcpad와 buffer 전달
7. pad의 peer인 output_sink의 chain 함수를 호출 (<span class="fill_color">output_chain_time</span>)


_output_chain_time_ 함수
```c
static guint buffer_count; // default 0

static GstFlowReturn
output_chain_time (GstPad * pad, GstObject * parent, GstBuffer * buffer)
{
  GstClockTime timestamp;
  guint8 b;

  timestamp = GST_BUFFER_TIMESTAMP (buffer);
  fail_unless_equals_int64 (timestamp,
      (buffer_count % N_BUFFERS) * 25 * GST_MSECOND);
  timestamp =
      gst_segment_to_stream_time (&current_segment, GST_FORMAT_TIME, timestamp);
  fail_unless_equals_int64 (timestamp,
      (buffer_count % N_BUFFERS) * 25 * GST_MSECOND);

  timestamp = GST_BUFFER_TIMESTAMP (buffer);
  timestamp =
      gst_segment_to_running_time (&current_segment, GST_FORMAT_TIME,
      timestamp);
  fail_unless_equals_int64 (timestamp, buffer_count * 25 * GST_MSECOND);

  gst_buffer_extract (buffer, 0, &b, 1);
  fail_unless_equals_int (b, buffer_count % N_BUFFERS);

  buffer_count++;
  gst_buffer_unref (buffer);
  return GST_FLOW_OK;
}
```

<br>

#### 3-3-3) EOS 전달

```c
    gst_pad_send_event (pad, gst_event_new_eos ());
```
&nbsp; 위에서 stream event를 새로 만든 것처럼 <span class="fill_color">gst_event_new_eos</span> 함수를 이용해 EOS event를 만들어서 <span class="fill_color">gst_pad_send_event</span> 함수를 호출한다. 아까와 마찬가지로 pad의 event 함수인 <span class="fill_color">gst_concat_sink_event</span> 함수가 호출되는데, 여기서 EOS 관련 부분이 실행된다. (함수들은 switch로 경우가 나눠져있음)






정리하면,  
1. <span class="fill_color">push_buffers_time</span> 에서 event 보내기
2. <span class="fill_color">gst_pad_chain</span> 으로 buf 보내기
3. 함수 내부에서 pad의 parent인 concat을 가져옴
4. concat의 <span class="fill_color">gst_concat_sink_event</span> 를 호출
5. <span clss="fill_color">gst_concat_switch_pad</span>를 호출하여 남은 pad가 있는지 확인

&nbsp; __<남은 pad가 있을 때>__

6. 다음 pad로 current_pad 가 바뀌고, duration이 바뀌었다는 message를 보낸다.

![concat_element7](https://bleetoteelb.github.io/assets/img/concat_element7.png)

<br>

&nbsp; __<남은 pad가 없을 때>__

&nbsp; 여기서 <span class="fill_color">gst_pad_push_event</span>가 실행되는데, push_data와 마찬가지로 src의 peer pad인 output_sink의 event 함수(<span class="fill_color">output_event_time</span>)을 호출한다.

6.  <span class="fill_color">gst_pad_push_event</span> 로 srcpad와 event 전달
7. <span class="fill_color">gst_pad_push_event_unchecked</span> & <span class="fill_color">gst_pad_send_event_unchecked</span> 로 srcpad와 event 전달
8. pad의 peer인 output_sink의 event 함수를 호출 (<span class="fill_color">output_event_time</span>)

![concat_element8](https://bleetoteelb.github.io/assets/img/concat_element8.png)

<br>

<br>

진짜로 그렇게 작동하는지 실제로 dot graph를 통해 확인해 보았다.
```c
  //GST_START_TEST 
  GstElement *pipeline = gst_pipeline_new("TestPipeline");

  GST_DEBUG_BIN_TO_DOT_FILE(GST_BIN(pipeline),GST_DEBUG_GRAPH_SHOW_ALL, "pipeline1");
  g_thread_join (thread1);
  GST_DEBUG_BIN_TO_DOT_FILE(GST_BIN(pipeline),GST_DEBUG_GRAPH_SHOW_ALL, "pipeline2");
  g_thread_join (thread2);
  GST_DEBUG_BIN_TO_DOT_FILE(GST_BIN(pipeline),GST_DEBUG_GRAPH_SHOW_ALL, "pipeline3");
  g_thread_join (thread3);
  GST_DEBUG_BIN_TO_DOT_FILE(GST_BIN(pipeline),GST_DEBUG_GRAPH_SHOW_ALL, "pipeline4");

```

<center>~ thread1</center>
<center><img src="https://bleetoteelb.github.io/assets/img/dotgraph_concat3.png" width="700"></center>
&nbsp; 
<center>thread1 ~ thread2</center>
<center><img src="https://bleetoteelb.github.io/assets/img/dotgraph_concat4.png" width="700"></center>
&nbsp;
<center>thread2 ~ thread3</center>
<center><img src="https://bleetoteelb.github.io/assets/img/dotgraph_concat5.png" width="700"></center>
&nbsp;
<center>thread3 ~ </center>
<center><img src="https://bleetoteelb.github.io/assets/img/dotgraph_concat6.png" width="700"></center>



## 4) Playbin3 에서 사용 예
* * *
__GOAL__ : Playbin3 에서 concat을 구성하여 사용하고 있다. concat 이 어느 포지션에 구성되는지 닷 그래프를 통해 확인해 보고 어떤 역할을 담당하는지 확인해 본다.
서버에 있는 아래 파일을 gst-play 를 통해 구동시켜보고 오디오 트랙을 변경하면 concat 에서 무슨 일들이 벌어지는지 확인해 본다.

ftp://10.186.118.224/guest/salmon/multi_audio/multi_audio.mkv (user / user), (http://10.186.118.224:5000/)

```bash
  cd gst-plugins-base/tools/        // base plugin 의 tools 폴더로 이동
  
  export USE_PLAYBIN3=1             // playbin3 를 쓰도록 환경변수 설정, 설정하지 않으면 Playbin2 를 기본적으로 사용
  
  ./gst-play-1.0 multi_audio.mkv    // 미디어 uri 주고 실행
  
  k                                 // k 입력하여 키보드 명령어 확인
  
  Interactive mode - keyboard controls:

	space    : pause/unpause
	q or ESC : quit
	> or n   : play next
	< or b   : play previous
	→        : seek forward
	←        : seek backward
	↑        : volume up
	↓        : volume down
	+        : increase playback rate
	-        : decrease playback rate
	d        : change playback direction
	t        : enable/disable trick modes
	a        : change audio track
	v        : change video track
	s        : change subtitle track
	0        : seek to beginning
	k        : show keyboard shortcuts

  0:00:02.7 / 0:02:09.1

  a                                 // a 입력하여 오디오 트랙 변경
```

잘 모르겠다...

![dotgraph_concat7](https://bleetoteelb.github.io/assets/img/dotgraph_concat7.png)