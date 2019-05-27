---
layout: post
title:  "Advanced GStreamer concepts(4)"
subtitle:   "Clocks and synchronization in GStreamer"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Clocks and synchronization in Gstreamer__

&nbsp; 복잡한 미디어를 재생할 때, 각 소리와 영상 샘플은 특정 시간에 특정 순서로 재생되어야 합니다. 이를 위해, Gstreamer는 동기화 체계를 제공합니다.

&nbsp; Gstreamer는 다음의 경우를 지원합니다.

- 재생 속도보다 빠른 접근이 가능한 라이브가 아닌 source. 이는 한쪽에서는 파일로부터 미디어를 읽어들이고 동기화된 방식으로 이를 재생하는 경우입니다. 이 경우에는 오디오, 비디오, 자막과 같은 여러 스트림들이 동기화 되어야 합니다.

- 여러 라이브 source에서 미디어를 캡쳐하고 동기화하여 혼합하고 압축. 오디오와 비디오를 마이크와 카메라로 녹음하고 저장을 위해 파일로 압축하는 전형적인 경우입니다.

- 버퍼링을 가진 느린 네트워크의 스트리밍. HTTP를 이용한 스트리밍 서버로부터 받은 데이터에 접근하는 전형적인 웹 스트리밍의 경우입니다.

- 라이브 source로부터 캡처하고 설정가능한 대기시간을 가진 라이브 source로 재생. 카메라로부터 source를 캡쳐하고 효과를 입혀서 그 결과물을 출력하는 경우입니다. UDP 네트워크를 사용하여 지연시간이 짧은 데이터를 스트리밍 할 때 사용됩니다.

- 사전 녹화된 컨텐츠를 라이브로 즉시 캡처하고 재생. 이전에 녹음된 오디오를 재생하고 새로운 오디오 샘플을 저장하는 오디오 녹음의 경우이며, 이전 데이터와 완벽하게 동기화 된 새로운 오디오를 얻는것이 목적입니다.

Gstreamer는 스트림의 동기화를 위하여 파이프라인에서 __GstClock__ 객체, 버퍼 타임스탬프와 SEGMENT event를 사용하며, 이는 다은 파트에서 우리가 알아볼 것입니다.

## __1. Clock running time__

&nbsp; 일반적인 컴퓨터에는, 시스템 시간, 사운드카드 시간, CPU 동작 시작 등등, 시간 자원으로 쓸 수 있는 많은 source들이 있기 때문에, Gstreamer에도 많은 적용가능한 __GstClock__ 들이 있습니다. 시계의 시간은 0이나 다른 특정한 값에서 시작할 필요는 없습니다. 어떤 시계는 특정한 시작날, 마지막 재시작 시각 등과 같은 기준을 잡기도 합니다.

&nbsp; __GstClock__ 은 <span class="fill_color">gst_clock_get_time()</span>의 시계에 따르는 절대시간을 반환합니다. 시계의 절대시간(혹은 clock time)은 단적으로 증가합니다.

