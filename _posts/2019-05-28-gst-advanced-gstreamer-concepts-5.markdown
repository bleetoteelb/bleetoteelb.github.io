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

- 버퍼 element는 높은 워터마크가 올 때까지 BUFFERING 메세지를 게시하고 있습니다. 이는 application은 파이프라인을 PAUSED상태로 있도록 지시하며, 그 상태는 결국에 sink에서 data가 preroll되는 동안 srcpad가 데이터를 밀어넣지 못하도록 막는 상태입니다.

- 만약 높은 워터마크가 온다면, BUFFERING 메세지는 100퍼센트를 게시한다는 것인데, 이는 application이 재생을 계속하도록 지시합니다.