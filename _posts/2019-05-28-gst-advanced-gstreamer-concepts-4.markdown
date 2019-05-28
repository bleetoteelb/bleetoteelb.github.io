---
layout: post
title:  "Advanced GStreamer concepts - Clocks and synchronization in Gstreamer"
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

Gstreamer는 스트림의 동기화를 위하여 파이프라인에서 __GstClock__ 객체, 버퍼 timestamp와 SEGMENT event를 사용하며, 이는 다음 파트에서 우리가 알아볼 것입니다.

## __1. Clock running time__

&nbsp; 일반적인 컴퓨터에는, 시스템 시간, 사운드카드 시간, CPU 동작 시작 등등, 시간 자원으로 쓸 수 있는 많은 source들이 있기 때문에, Gstreamer에도 많은 적용가능한 __GstClock__ 들이 있습니다. 시계의 시간은 0이나 다른 특정한 값에서 시작할 필요는 없습니다. 어떤 시계는 특정한 시작날, 마지막 재시작 시각 등과 같은 기준을 잡기도 합니다.

&nbsp; __GstClock__ 은 <span class="fill_color">gst_clock_get_time()</span>의 시계에 따르는 절대시간(absolute-time)을 반환합니다. 시계의 절대시간(혹은 clock time)은 단조롭게 증가합니다.

&nbsp; 실행시간(running-time)은 기준시간(base-time)이라 불리는 절대시간의 이전 스냡샷과 다른 절대시간 간의 차이를 의미합니다.
```
running-time = absolute-time - base-time
```

&nbsp; Gstreamer __GstPipeline__ 객체는 __GstClock__ 객체와 파이프라인이 PLAYING 상태로 갈 때의 기준시간을 포함합니다. 선택된 기준 시간과 함께 파이프라인 속 각 element에게 파이프라인은 선택된 __GstClock__ 의 처리를 제공합니다. 파이프라인은 실행시간이 PLAYING 상태에서 흐른시간을 기준시간이 반영하는 방식으로 기준시간을 선택합니다. 결과적으로, 파이프라인이 PAUSED일때, 실행시간 역시 멈춥니다.

&nbsp; 파이프라인 속 모든 객체들이 간은 시계와 기준시간을 갖고 있기 때문에, 그들은 파이프라인 시계에 따라 실행시간을 계산합니다.

## __2. Buffer running-time__
&nbsp; 버퍼의 실행시간을 계산하기 위해, 우리는 버퍼의 timestamp와 버퍼 이전의 SEGMENT event가 필요합니다. 먼저 우리가 SEGMENT event를 __GstSegment__ 객체로 변환하면 <span class="fill_color">gst_segment_to_running_time()</span>함수를 이용하여 버퍼의 실행시간을 계산해낼 수 있습니다.

&nbsp; 이제 특정한 실행시간을 가진 버퍼를 시스템의 시계가 버퍼와 같은 실행시간을 가질 때 재생되도록 하는것으로 동기화를 할 수 있습니다. 보통 이 과정은 sink element에서 진행됩니다. 또한 이 element는 파이프라인 시계에 동기화하기 전에 구성된 파이프라인의 지연 시간을 고려하여 이를 버퍼 실행시간에 추가해야합니다.

&nbsp; 라이브가 아닌 source의 버퍼의 timestamp는 실행시간이 0부터 시작합니다. 탐색을 한 후에는, 버퍼의 실행시간을 0부터 다시 시작합니다.

&nbsp; 라이브 source는 버퍼의 첫 데이터가 들어왔을 때, 파이프라인의 실행시간과 맞는 실행시간을 가진 buffer의 timestamp가 필요합니다.


## __3. Buffer stream-time__

&nbsp; 스트림에서의 위치라고 알려진 버퍼의 스트림시간은 0과 미디어의 총 길이 사이의 값이며, 이전 SEGMENT event와 버퍼의 timestamp 로부터 계산됩니다.

&nbsp; 스트림 시간은 다음과 같은 곳에서 사용됩니다.

- __POSITION__ query를 가진 스트림의 현재 위치를 보고할 때.
- 탐색 event와 query에 사용되는 위치
- 동기화 제어 값으로 사용되는 위치

&nbsp; 스트림 시간은 동기화 스트림에는 절대 사용되지 않고, 실행시간에만 사용됩니다.

## __4. Time overview__

&nbsp; Gsstreamer에서 사용되는 다양한 타임라인들의 개요를 정리해 보았습니다.

&nbsp; 아래 그림에서는 파이프라인에서 100ms 샘플을 재생하고 50ms~100ms 사이의 부분을 반복할 때 다른 시간들을 표시하였습니다.

![clocks](https://bleetoteelb.github.io/assets/img/clocks.png)

&nbsp; 위 그림을 통해 버퍼의 실행시간이 어떻게 항상 시계를 따라 단조롭게 증가하는지를 알 수 있습니다. 버퍼들은 그들의 실행시간이 "clock-time - base-time" 과 같을 때 실행됩니다. 스트림 시간은 스트림에서의 위치를 가리키고 구간 반복을 할 때 값이 반복의 시작 위치로 돌아갑니다.

## __5. Clock providers__

&nbsp; 시계 제공자는 __GstClock__ 객체를 제공할 수 있는 파이프라인의 element입니다. 시계 객체는 element가 PLAYING 상태일 때 단조롭게 증가하는 절대 시간을 알릴 때 필요합니다. element가 PAUSED일 때 시계 역시 일시정지 할 수 있습니다.

&nbsp; 시계 제공자는 특정한 속도로 미디어를 재생하기 때문에 존재하는데, 이 속도가 시스템의 시계와 같은 속도일 필요는 없습니다. 예를들어, 사운드카드가 44.1 KHz로 재생을 하고 있는데, 그것이 시스템 시계에 따라 정확하게 1초 후 라는 뜻은 아니지만 사운드 카드가 44.100 샘플들을 재생하고 있는 것입니다. 추정에 의해서만 맞는 말이고, 사실은 오디오 장치가 노출할수 있는 재생 샘플의 수에 따른 내부 시계를 갖고있습니다.

&nbsp; 만약 내부 시계를 가진 element가 동기화가 필요하다면, 파이프라인 시계에 따른 시간이 내부 시계에 따라 언제 발생할지 추정할 필요가 있습니다.

&nbsp; 파이프라인 시계가 정확하게 element의 내부시계와 같다면, element는 slaving 단계를 뛰어넘고 재생 일정을 정하기 위해 파이프라인 시계를 바로 쓸 수 있을 것입니다. 이것이 더 빠르고 정확한 방법일 수 있습니다. 그러므로 일반적으로 오디오 입력이나 출력 장치 같은 내부시계가 있는 element는 파이프라인에서 시계 제공자가 됩니다.

&nbsp; 파이프라인이 PLAYING 상태가 되면, sink부터 source까지 모든 파이프라인의 element의 상태가 PLAYING이 되면서 각 element에게 시계를 제공하도록 요청합니다. 시계를 제공할 수 있는 마지막 element는 파이프라인에서 시계 제공자로 사용됩니다. 이 알고리즘은 전형적인 재생 파이프라인 속 오디오 sink의 시계와 전형적인 캡처 파이프라인 속 source element의 시계를 선호합니다.

&nbsp; 파이프라인에는 시계와 시계 공급자에 대해 알려주기 위한 bus 메세지가 있습니다. 파이프라인에서 어떤 시계가 선택되었는지를 bus의 __NEW_CLOCK__ 메세지를 통해 확인할 수 있습니다. 파이프라인에서 시계 공급자가 제거되면, __CLOCK_LOST__ 메세지가 게시되어, application이 PAUSED가 되어야만 하고 새로운 시계를 선택하기 위해 PLAYING 상태로 돌아갑니다.

## __6. Latency__

&nbsp; latency는 timestamp X에서 캡쳐된 샘플이 sink에 도달하는 데 걸리는 시간입니다. 이 시간은 파이프라인의 시계와 비교하여 측정됩니다. 시계와 동기화 하는 유일한 element가 sink인 파이프라인에서, 다른 element가 버퍼를 지연시키지 않으므로 latency는 항상 0입니다.

&nbsp; 라이브 source를 가지는 파이프라인에서는, 대부분 live source가 동작하는 방식 때문에 latency가 발생합니다. 오디오 source를 생각해보면, 시간이 0인 첫번째 샘플을 캡처하기 시작합니다. 만약 44100 Hz에서 source가 버퍼를 44100 샘플씩 밀어넣는다면, 1초가 되었을 때 버퍼가 받아질 것입니다. 버퍼의 timestamp는 0인 반면 시계의 시간은 1초 이상 흘렀기 때문에, sink는 지나간 버퍼를 버릴 것입니다. sink에서 지연 보상이 없다면 모든 버퍼는 버려질 것입니다.

### __6.1. Latency compensation__

&nbsp; 파이프라인이 PLAYING 상태로 가기 전에, 기준시간을 계산하고 시계를 선택하는것과 더불어 파이프라인 안에서 지연을 계산할 것입니다. 이 과정은 파이프라인 안의 모든 sink에서 LATENCY query를 통해 이뤄집니다. 파이프라인은 최대 지연시간을 파이프라인에서 선택하고 이를 LATENCY event에 설정합니다.

&nbsp; 모든 sink element는 LATENCY event 안의 값 만큼 재생을 지연합니다. 모든 sink가 같은 시간만큼 지연하기 때문에, 상대적인 동기화가 이루어질 것입니다.


### __6.2. Dynamic Latency__

&nbsp; 파이프라인에/에서 element를 추가/제거 하거나 element 속성을 바꾸는 것은 파이프라인에서 지연을 바꿀 수 있습니다. element는 bus에 LATENCY 메세지에 연결함으로써 latency 변경을 파이프라인에 요청할 수 있습니다. application은 새 latency를 query하고 재분배 할것인지를 결정할 수 있습니다. 파이프라인에서 latency를 변경하는것은 시각적 청각적 결함이 발생할 수 있으므로 허용된 경우에만 application이 수행해야 합니다.