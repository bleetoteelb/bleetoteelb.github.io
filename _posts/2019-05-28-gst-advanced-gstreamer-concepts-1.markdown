---
layout: post
title:  "Advanced GStreamer concepts - Position tracking and seeking"
subtitle:   "Position tracking and seeking"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>


# __Position tracking and seeking__

&nbsp; 지금까지 미디어 처리를 위해 어떻게 파이프라인을 만들고 어떻게 실행하는지에 대해 알아보았습니다. 대부분의 application 개발자들은 미디어 처리과정에 대해 유저에게 표시해 주는 것에 관심을 갖고 있습니다. 예를들어 미디어 플레이어는 노래의 진행 상황을 보여주는 슬라이드와 스트림 길이를 가리키는 라벨을 보여주길 원합니다. 또, 코딩 변환 application은 작업이 어느정도 진행되었는지 알려주는 진행 표시줄을 보여주길 원할 것입니다. Gstreamer는 이러한 작업들을 _querying_이라고 알려진 개념을 이용해서 지원합니다. 이것과 탐색(Seeking)은 매우 비슷하기 때문에, event의 개념을 이용하는 탐색 역시 이 파트에서 다루도록 하겠습니다.

## __1. Querying: getting the position or length of a stream__

&nbsp; querying은 진행률 추적과 관련된 특정 스트림 속성을 요청하는 것으로 정의됩니다. 이는 가능하다면 stream의 길이와 현재 위치를 얻는 것을 포함합니다. 그러한 스트림 속성들은 시간, 오디오샘플, 비디오 프레임, 바이트 등 다양한 타입으로 존재합니다. 몇가지 편리한 다른 함수들 역시 제공되지만 (예를들어, <span class="fill_color">gst_element_query_position()</span> 혹은 <span class="fill_color">gst_element_query_duration()</span> )가장 보편적으로 사용되는 함수는 <span class="fill_color">gst_element_query()</span>입니다. 일반적으로 파이프라인에 직접 query를 보낼 수 있으며, 어떤 element를 query할 것인지 등 사용자 내부의 세부 정보를 파악할 수 있습니다.

&nbsp; 내부적으로, query는 sink로 보내지며 하나의 element가 처리할 수 있을 때까지 거꾸로 "분할" 될 것입니다. 그 결과는 함수를 호출한 곳으로 되돌려 보내집니다. 일반적으로 source가 그 자체인 웹캠의 live source일지라도 demuxer에서 이런 과정이 일어난다.

```c
#include <gst/gst.h>

static gboolean
cb_print_position (GstElement *pipeline)
{
  gint64 pos, len;

  if (gst_element_query_position (pipeline, GST_FORMAT_TIME, &pos)
    && gst_element_query_duration (pipeline, GST_FORMAT_TIME, &len)) {
    g_print ("Time: %" GST_TIME_FORMAT " / %" GST_TIME_FORMAT "\r",
         GST_TIME_ARGS (pos), GST_TIME_ARGS (len));
  }

  /* call me again */
  return TRUE;
}

gint
main (gint   argc,
      gchar *argv[])
{
  GstElement *pipeline;

[..]

  /* run pipeline */
  g_timeout_add (200, (GSourceFunc) cb_print_position, pipeline);
  g_main_loop_run (loop);

[..]

}
```

## __2. Events: seeking (and more)__

&nbsp; Event는 query와 매우 비슷한 방식으로 동작합니다. 예를들어, Dispatching은 event와 정확히 똑같이 동작하며(심지어 제한도 같습니다) 그들은 최고레벨의 파이프라인에 비슷하게 보내지고 당신을 위해 모든것을 알아낼 것입니다. 비록 application과 element가 event를 사용해 상호작용하는 더 많은 방법들이 있지만, 우리는 여기서 탐색(seeking)에만 초점을 맞출 것입니다. 이는 seek-event를 이용해 이루어지는데, seek-event에는 재생속도, 탐색 오프셋 형식(시간, 오디오샘플, 비디오 프레임, 바이트 같이 오프셋의 단위가 따라야 하는 형식),선택적으로 탐색과 관련된 플래그(내부 버퍼가 플러시 되어야 하는지 여부 등), 탐색방법(주어진 오프셋과 비교하여 표시하는 방법), 그리고 탐색 오프셋이 포함됩니다. 첫번째 오프셋(현재 위치)는 찾아야할 새로운 위치인 반면, 두번째 오프셋(정지)는 선택적이며 스트리밍이 언제 멈춰야 하는지를 구분해줍니다. 보통 __GST_SEEK_TYPE_NONE__ 와 __-1__ 을 종료 위치로 구분하는 것이 좋습니다. 탐색 기능은 <span class="fill_color">gst_element_seek()</span> 에서 처리됩니다.

```c
static void
seek_to_time (GstElement *pipeline,
          gint64      time_nanoseconds)
{
  if (!gst_element_seek (pipeline, 1.0, GST_FORMAT_TIME, GST_SEEK_FLAG_FLUSH,
                         GST_SEEK_TYPE_SET, time_nanoseconds,
                         GST_SEEK_TYPE_NONE, GST_CLOCK_TIME_NONE)) {
    g_print ("Seek failed!\n");
  }
}
```

&nbsp; __GST_SEEK_FLAG_FLUSH__ 를 가진 탐색은 파이프라인이 PAUSED 혹은 PLAYING 상태일때 수행되어야 합니다. 탐색 후 들어오는 새로운 데이터가 파이프라인을 다시 예열할 때까지 파이프라인은 자동으로 예열상태로 전환된다. 파이프라인이 예열 되면, 이는 다시 탐색이 실행되었던 때의 상태인 PAUSED 혹은 PLAYING 상태로 되돌아갑니다. 탐색이 실행되면 그것이 완료될 때까지 <span class="fill_color">gst_element_get_state()</span>를 통해 기다리거나(blocking) bus에서 전달하는 __ASYNC_DONE__ 메세지를 기다릴 수 있습니다.

&nbsp; __GST_SEEK_FLAG_FLUSH__ 가 없는 탐색은 파이프라인이 PLAYING 상태일 때만 수행되어야 합니다. 플러쉬가 없는 탐색이 PAUSED 상태에서 실행되면 sink에서 스레드를 스트리밍하고 있는 파이프라인이 block되기 때문에 교착상태(deadlock)에 빠질 수 있습니다.

&nbsp; <span class="fill_color">gst_element_seek()</span> 함수에서 return이 될 때 탐색이 끝나기 때문에 탐색은 즉시 일어나지 않는다는것을 꼭 알아두어야 합니다. 관련된 특정 element에 따라 실제 탐색은 나중에 다른 스레드(스트리밍 스레드)에서 수행 될 수 있으며, 새로운 탐색 위치의 버퍼가 sink와 같은 downstream element에 도달 할 때까지 약간의 시간이 걸릴 수 있습니다.(탐색이 비플러쉬일 경우 시간이 조금 더 걸릴 수도 있습니다)

&nbsp; 슬라이더 이동에 대한 직접 대응처럼 다수의 탐색을 짧은 간격으로 수행할수도 있습니다. 탐색후에 내부적으로 파이프라인은 일시 정지되고(재생 중이면), 위치는 내부적으로 재설정 되어 demuxer와 decoder는 새로운 위치부터 계속 디코딩하고  이것은 모든 sink가 데이터로 가득 차면 다시 시작할것입니다. 만약 제대로 재생이 된다면, 파이프라인은 playing 상태로 또다시 설정될 것입니다. 새로운 위치가 비디오 출력에서 즉시 가능해지기 때문에, 파이프라인이 재생상태가 아닐지라도 새로운 프레임을 볼 수 있을 것입니다.