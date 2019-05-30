---
layout: post
title:  "Advanced GStreamer concepts - Buffering"
subtitle:   "Buffering"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Buffering__

&nbsp; 버퍼링의 목적은 파이프라인에 충분한 데이터를 축적하여 재생이 끊김없이 부드럽게 되도록 함에 있습니다. 이는 보통 느리고 라이브가 아닌 네트워크 source에서 데이터를 받을 때 사용되지만 라이브 source에서도 사용될 수 있습니다.

&nbsp; Gstreamer는 다음의 기능들을 제공합니다.

- 재생을 시작하기 전에 메모리에 일정한 양의 데이터만큼 버퍼를 채워서 네트워크 변동을 최소화합니다. 아래의 __Stream buffering__ 부분을 참조하십시오.

- 네트워크 파일을 로컬 디스크에 저장하여 다운로드된 데이터에서 빠르게 탐색하도록 합니다. 이는 quicktime/youtube player와 비슷합니다. 아래의 __Download buffering__ 부분을 참조하십시오.

- 캐시 영역에서 탐색하는, 디스크의 로컬 ringbuffer(역주: 머리와 끝이 고리와 같이 이어져 있다고 생각하는 버퍼)에 반라이브 스트림을 임시로 저장합니다. 이는 tivo(역주: 하드디스크에 tv 프로그램을 자동 녹화하는 장치)와 유사한 timeshifting입니다. 아래의 __Timeshift buffering__ 부분을 참조하십시오.

&nbsp; Gstreamer는 application에 현재 버퍼링 상태에 대해 진행 상황을 알려줄 뿐만 아니라 application이 어떻게 버퍼링을 할 것인지와 언제 버퍼링을 멈출 것인지를 결정할 수 있도록 합니다.

&nbsp; 대부분의 경우에, application은 bus에 있는 BUFFERING 메세지를 받고있어야 합니다. BUFFERING 메세지 내의 진행 비율이 100보다 작다면, 파이프라인은 버퍼링 중입니다. 메세지가 100퍼센트에 도달하면, 버퍼링은 완료된 것입니다. 버퍼링 상태에서 application은 파이프라인을 PAUSED 상태로 두어야 합니다. 버퍼링이 완료됐을 때, 파이프라인이 PLAYING 상태로 돌려놓습니다.

&nbsp; 아래 코드는 메세지 핸들러가 어떻게 BUFFERING 메세지를 처리하는지에 대한 예시입니다. 이에 대한 더 자세한 내용은 아래의 __Buffering strategies__ 부분에서 확인할 것입니다.

```c
  [...]

  switch (GST_MESSAGE_TYPE (message)) {
    case GST_MESSAGE_BUFFERING:{
      gint percent;

      /* no state management needed for live pipelines */
      if (is_live)
        break;

      gst_message_parse_buffering (message, &percent);

      if (percent == 100) {
        /* a 100% message means buffering is done */
        buffering = FALSE;
        /* if the desired state is playing, go back */
        if (target_state == GST_STATE_PLAYING) {
          gst_element_set_state (pipeline, GST_STATE_PLAYING);
        }
      } else {
        /* buffering busy */
        if (!buffering && target_state == GST_STATE_PLAYING) {
          /* we were not buffering but PLAYING, PAUSE  the pipeline. */
          gst_element_set_state (pipeline, GST_STATE_PAUSED);
        }
        buffering = TRUE;
      }
      break;
    case ...

  [...]
```

## __1. Stream buffering__

```
      +---------+     +---------+     +-------+
      | httpsrc |     | buffer  |     | demux |
      |        src - sink      src - sink     ....
      +---------+     +---------+     +-------+

```

&nbsp; 이 경우에 느린 네트워크 source로부터 buffer element로 데이터를 읽는 경우입니다. (queue2와 같이)

&nbsp; 버퍼 element는 바이트에 나타난 낮고 워터마크와 높은 워터마크를 갖고있습니다. 버퍼는 워터마크를 다음과 같이 사용합니다.

- 버퍼 element는 높은 워터마크가 발생 할 때까지 BUFFERING 메세지를 게시하고 있습니다. 이는 application은 파이프라인을 PAUSED상태로 있도록 지시하며, 그 상태는 결국에 sink에서 data가 preroll되는 동안 srcpad가 데이터를 밀어넣지 못하도록 막는 상태입니다.

- 만약 높은 워터마크가 발생한다면, BUFFERING 메세지는 100퍼센트를 게시한다는 것인데, 이는 application이 재생을 계속하도록 지시합니다.

- 재생 도중 낮은 워터마크가 발생하면 대기열에서 BUFFERING 메세지를 다시 게시하기 시작하고, 높은 워터마크가 발생할 때까지 application이 파이프라인을 PAUSE 상태로 만들도록 합니다. 이를 재버퍼링 상태라고 부릅니다.

- 재생 도중, 대기열 수준은 네트워크 불규칙성을 보완하기 위한 방법으로 높은 워터마크와 낮은 워터마크 사이에서 변동할 것입니다.

&nbsp; 이 버퍼링 방법은 push 모드에서 demuxer가 작동할 때 사용 가능한 방법입니다. 스트림을 탐색하려면 네트워크 source에서 검색이 필요합니다. 이러한 방법은 라이브 스트리밍이나 효율적인 검색이 불가능/불필요한 경우처럼 파일의 총 길이를 알 수 없을 때 적합합니다.

&nbsp; 낮은 워터마크나 높은 워터마크를 잘 설정하는 것이 중요합니다. 이를 설정하는 몇가지 다음과 같은 아이디어들이 있습니다.

- 네트워크 대역폭을 측정하고 버퍼링에 걸리는 일정한 시간에 따라 워터마크의 높낮음을 설정합니다.  
Gstreamer core의 queue2 element에서 max-size-time와 use-rate-estimate 속성을 추가하여 그 역할을 하도록 합니다. 또한 재생시 buffer-duration 속성은 속도를 추정하여 버퍼된 데이터의 양을 조절합니다.

- 코덱의 bitrate에 따라 재생이 시작하기 전에 일정한 데이터를 버퍼하도록 워터마크를 설정하는 것도 가능합니다.

- 재버퍼링 사이의 시간이 application에서 설정한 제한이 되기 전까지 대기열의 크기를 증가시키고 재버퍼링 사이의 시간을 측정하고 고정된 byte로 시작할 수도 있습니다.

&nbsp; 버퍼링 element는 파이프라인 어디든 들어갈 수 있습니다. 예를들어 버퍼링 element가 decoder 전에 넣을 수 있습니다. 이는 시간에 따라 높고 낮은 워터마크를 설정할 수 있도록 해줍니다.

&nbsp; playbin에 있는 버퍼링 플래그는 분석된 데이터에서 버퍼링을 수행합니다. 버퍼링의 또 다른 장점은 demuxer가 pull 모드로 동작하도록 할 수 있습니다. 이는 느린 네트워크 드라이버(filesrc를 가진)에서 데이터를 읽어들일 때 버퍼할 수 있는 좋은 방법입니다.

## __2. Download buffering__

```
      +---------+     +---------+     +-------+
      | httpsrc |     | buffer  |     | demux |
      |        src - sink      src - sink     ....
      +---------+     +----|----+     +-------+
                           V
                          file
```

&nbsp; 만약 서버가 고정된 길이의 파일을 클라이언트에게 스트리밍하는 것을 알 수 있다면, application은 디스크의 전체 파일을 다운로드 하게 할 수 있습니다. buffer element는 다운로드 된 파일을 텀색하기 위해 demuxer에 push 혹은 pull 기반의 srcpad를 제공합니다.

&nbsp; 이 모드는 클라이언트가 서버의 파일 길이를 알 수 있을 때만 적합합니다.

&nbsp; 이 경우에, 버퍼링 메세지는 주로 요구된 범위가 (다운로드 된 영역+버퍼사이즈)내에 없을 경우 발생합니다. 버퍼링 메세지는 증분 다운로드가 수행되고 있다는 표시도 포함합니다. 이 플래그는 BUFFERING query를 사용하여 application이 버퍼링이 더 지능적으로 제어하는데에 사용됩니다. 아래의 __Buffering strategies__ 에서 자세한 내용을 참고하십시오.

## __3. Timeshift buffering__

```
      +---------+     +---------+     +-------+
      | httpsrc |     | buffer  |     | demux |
      |        src - sink      src - sink     ....
      +---------+     +----|----+     +-------+
                           V
                       file-ringbuffer
```
&nbsp; 이 모드에서, 고정 크기의 링버퍼는 서버 내용을 계속 다운로드합니다. 이는 버퍼 데이터에서 탐색을 할 수 있도록 해줍니다. 링버퍼의 크기에 따라 시간을 거슬러 올라갈 수도 있습니다.

&nbsp; 이 모드는 모든 실시간 스트림에서 적합합니다. 증분 다운로드 모드와 같이, 버퍼링 메세지는 timeshifting 다운로드가 진행중이라는 표시와 함께 발생합니다.

##  __4. Live buffering__

&nbsp; 실시간 파이프라인에서 일반적으로 캡처 element와 재생 element 사이에 고정된 지연시간을 설정합니다. 이 지연시간은 jitterbuffer와 같은 대기열이나 audiosink의 다른 수단에 의해 설정할 수 있습니다.

&nbsp; 버퍼링 메세지는 실시간 파이프라인에서도 발생할 수 있으며, 사용자에게 지연 버퍼링 표시 역할을 합니다. application은 보통 상태변경을 동반하는 이러한 버퍼링 메세지에 반응하지 않습니다.

## __5. Buffering strategies__

&nbsp; 이제부터는 버퍼링 메세지와 버퍼링 쿼리에 기반한 다른 버퍼링 전략들을 적용하는 아이디어들을 소개합니다.

### __5.1. No-rebuffer strategy__

&nbsp; 우리는 파이프라인에 충분한 데이터를 축적하여 재생이 끊김없도록 하고자 합니다. 이 목표를 달성하기 위해서 파일에서 남은 총 재생시간과 남은 다운로드 시간을 알아야 합니다. 만약 버퍼링 시간이 재생시간보다 적을경우, 우리는 끊김없이 재생할 수 있습니다.

&nbsp; 우리는 DURATION, POSITION 그리고 BUFFERING 쿼리들을 이용할 수 있습니다. 현재 버퍼링 상태를 얻으려면 주기적으로 버퍼링 쿼리를 실행해야 합니다. 우리는 최악의 경우에 온전한 파일을 저장할 수 있을 만큼의 큰 버퍼가 필요할 수 있습니다. 이 버퍼링 전략은 다운로드 버퍼링과 함께 사용하는 것이 가장 좋습니다. __Download buffering__ 을 참조하십시오.

&nbsp; 다음은 이와 관련된 예제코드입니다.

```c
#include <gst/gst.h>

GstState target_state;
static gboolean is_live;
static gboolean is_buffering;

static gboolean
buffer_timeout (gpointer data)
{
  GstElement *pipeline = data;
  GstQuery *query;
  gboolean busy;
  gint percent;
  gint64 estimated_total;
  gint64 position, duration;
  guint64 play_left;

  query = gst_query_new_buffering (GST_FORMAT_TIME);

  if (!gst_element_query (pipeline, query))
    return TRUE;

  gst_query_parse_buffering_percent (query, &busy, &percent);
  gst_query_parse_buffering_range (query, NULL, NULL, NULL, &estimated_total);

  if (estimated_total == -1)
    estimated_total = 0;

  /* calculate the remaining playback time */
  if (!gst_element_query_position (pipeline, GST_FORMAT_TIME, &position))
    position = -1;
  if (!gst_element_query_duration (pipeline, GST_FORMAT_TIME, &duration))
    duration = -1;

  if (duration != -1 && position != -1)
    play_left = GST_TIME_AS_MSECONDS (duration - position);
  else
    play_left = 0;

  g_message ("play_left %" G_GUINT64_FORMAT", estimated_total %" G_GUINT64_FORMAT
      ", percent %d", play_left, estimated_total, percent);

  /* we are buffering or the estimated download time is bigger than the
   * remaining playback time. We keep buffering. */
  is_buffering = (busy || estimated_total * 1.1 > play_left);

  if (!is_buffering)
    gst_element_set_state (pipeline, target_state);

  return is_buffering;
}

static void
on_message_buffering (GstBus *bus, GstMessage *message, gpointer user_data)
{
  GstElement *pipeline = user_data;
  gint percent;

  /* no state management needed for live pipelines */
  if (is_live)
    return;

  gst_message_parse_buffering (message, &percent);

  if (percent < 100) {
    /* buffering busy */
    if (!is_buffering) {
      is_buffering = TRUE;
      if (target_state == GST_STATE_PLAYING) {
        /* we were not buffering but PLAYING, PAUSE  the pipeline. */
        gst_element_set_state (pipeline, GST_STATE_PAUSED);
      }
    }
  }
}

static void
on_message_async_done (GstBus *bus, GstMessage *message, gpointer user_data)
{
  GstElement *pipeline = user_data;

  if (!is_buffering)
    gst_element_set_state (pipeline, target_state);
  else
    g_timeout_add (500, buffer_timeout, pipeline);
}

gint
main (gint   argc,
      gchar *argv[])
{
  GstElement *pipeline;
  GMainLoop *loop;
  GstBus *bus;
  GstStateChangeReturn ret;

  /* init GStreamer */
  gst_init (&amp;argc, &amp;argv);
  loop = g_main_loop_new (NULL, FALSE);

  /* make sure we have a URI */
  if (argc != 2) {
    g_print ("Usage: %s &lt;URI&gt;\n", argv[0]);
    return -1;
  }

  /* set up */
  pipeline = gst_element_factory_make ("playbin", "pipeline");
  g_object_set (G_OBJECT (pipeline), "uri", argv[1], NULL);
  g_object_set (G_OBJECT (pipeline), "flags", 0x697 , NULL);

  bus = gst_pipeline_get_bus (GST_PIPELINE (pipeline));
  gst_bus_add_signal_watch (bus);

  g_signal_connect (bus, "message::buffering",
    (GCallback) on_message_buffering, pipeline);
  g_signal_connect (bus, "message::async-done",
    (GCallback) on_message_async_done, pipeline);
  gst_object_unref (bus);

  is_buffering = FALSE;
  target_state = GST_STATE_PLAYING;
  ret = gst_element_set_state (pipeline, GST_STATE_PAUSED);

  switch (ret) {
    case GST_STATE_CHANGE_SUCCESS:
      is_live = FALSE;
      break;

    case GST_STATE_CHANGE_FAILURE:
      g_warning ("failed to PAUSE");
      return -1;

    case GST_STATE_CHANGE_NO_PREROLL:
      is_live = TRUE;
      break;

    default:
      break;
  }

  /* now run */
  g_main_loop_run (loop);

  /* also clean up */
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (GST_OBJECT (pipeline));
  g_main_loop_unref (loop);

  return 0;
}
```

&nbsp; 어떻게 파이프라인이 PAUSED 상태로 먼저 설정했는지 확인하십시오. 버퍼링이 필요할 때 우리는 버퍼링 메세지를 preroll단계에서 받을 것입니다. 우리가  preroll할 때 (on_message_async_done 함수에서), 우리는 버퍼링이 진행되고 있는지 확인하고, 아니라면 재생을 시작합니다. 버퍼링이 진행되고 있다면 버퍼링 상태를 등록하기 위해 시간초과를 시작합니다. 만약 추정된 다운로드 시간이 남은 재생시간보다 적을경우, 재생을 시작합니다.