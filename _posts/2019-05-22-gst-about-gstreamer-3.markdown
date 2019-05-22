---
layout: post
title:  "About Gstreamer(3)"
subtitle:   "Foundations"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Foundations__

&nbsp; 이 부분에서는 Gstreamer의 기본 개념을 소개할 것입니다. 이 개념을 이해하는것은 이 가이드의 나머지 부분에서 전제되는 사항이기 때문에 이 부분을 이해하는것은 굉장히 중요합니다.

## __1. Elements__

&nbsp; __*Element*__ 는 Gstreamer의 객체 클래스에서 가장 중요한 요소입니다. 당신은 보통 서로 연결된 element 체인을 생성하고 이 element 체일을 통해 데이터가 흘러가도록 합니다. element는 한가지 특수한 기능을 갖고 있는데, 데이터를 파일로부터 읽거나, 데이터를 디코딩하거나 사운드 카드로 데이터를 출력합니다. 이러한 element들을 서로 연결함으로써 당신은 특수한 역할을 하는 파이프 라인을 생성할 수 있습니다. ( 예를들어, 미디어 재생이나 캡처 ) Gstreamer 기본적으로 많은 element들이 포함되어 있어, 다양한 미디어 프로그램들을 개발 할 수 있게 해줍니다. 이 주제는 [GStreamer Plugin Writer's Guide](https://gstreamer.freedesktop.org/documentation/plugin-development/index.html?gi-language=c)에 자세히 설명되어 있습니다.

<br>

## __2. Pads__

&nbsp; *__Pad__* 는 다른 element와 링크할 수 있을 때 element의 입력과 출력입니다. Pad들은 Gstreamer의 element들 사이에서 링크과 데이터흐름을 다루는데 사용됩니다. Pad는 다른 element와 연결을 생성하는 element element에 "plug" 혹은 "port"된다고 볼수도 있는데, 이 element들로부터 혹은 이 element들로 데이터가 흐를 수 있습니다. Pad들은 이러한 pad를 통해 흐르는 데이터의 타입을 제한할 수 있는 특수한 기능을 갖고 있습니다. 두개의 Pad 양쪽에서 모두 허용된 데이터 타입이 호환 가능할 경우에만 링크가 허용됩니다. 데이터 타입은 "caps negotitation"이라 불리는 프로세스를 이용하여 pad들 사이에서 협상됩니다. 데이터 타입은 _GstCaps_ 에 있습니다.

&nbsp; 여기서 Pad의 이해를 돕기위해 비유를 해보겠습니다. pad는 물리적 장치의 plug 혹은 잭과 비슷합니다. 예를 들어, 오디오 앰프, DVD 플레이어, 비디오 프로젝터로 구성된 홈 시어터 시스템을 생각해 보십시오. DVD 플레이어에서 앰프로의 링크는 양쪽 장치에서 오디오 잭을 허용하기 때문에 가능한 것이고, 프로젝터로부터 DVD 플레이어로의 링크는 양쪽 장치에서 호환되는 비디오 잭이 있기 때문입니다. 프로젝터와 앰프 사이에서는 서로 다른 타입의 잭만을 허용하기 때문에 링크가 만들어지지 않을것입니다. Gstreamer의 pad는 홈시어터 시스템에서 잭과 같은 용도로 사용됩니다.

&nbsp; 대부분의 경우, Gstreamer의 모든 데이터들은 링크를 통해 한 방향으로 흐릅니다. 데이터들은 한 element에서 나와 하나 혹은 이상의 source pad를 통해 흐르고 element들이 하나 혹은 그 이상의 sink pad를 통해 데이터들을 받아들입니다. Source와 Sink element들에 각각 오직 source와 sink pad들만 있습니다. 데이터는 buffer들( _GstBuffer_ 객체에 있는 ) 혹은 event들( _GstEvent_ 객체에 있는 )을 의미합니다.

<br>

## __3. Bins and pipelines__

&nbsp; *__Bin__* 은 element들을 모아놓은 컨테이너입니다. bin들이 element 자체의 하위 클래스 이기 때문에, 당신은 bin을 대부분 마치 element인 것처럼 제어 할 수 있으므로 application에서 많은 복잡성을 추상화 할 수 있습니다. 예를들어, bin 자체의 상태를 변경 함으로써 bin속 모든 elements들의 상태를 변경할 수 있다. bin들은 또한 포함된 하위 messages들( 가령, error messages, tag messages, EOS messages )로부터 온 bus messages을 전달할 수 있다.

&nbsp; 파이프라인은 최상위 수준의 bin입니다. bin은 application에 bus를 전달하고 그 하위 요소들의 동기화를 관리합니다. 당신이 bin을 _PAUSED_ 혹은 _PLAYING_ 상태로 설정하면 데이터 흐름은 시작되고 미디어 데이터의 처리가 시작될 것입니다. 한번 시작되면 파이프라인들은 당신이 그것들을 멈추거나 데이터 스트림이 마지막에 도달할 때까지 별도의 스레드에서 실행됩니다.

![simple-player](https://bleetoteelb.github.io/assets/img/simple-player.png)

<br>

## __4. Communication__

&nbsp; Gstreamer는 application과 파이프라인 사이의 데이터 교환과 통신을 위해 여러가지 체계를 제공합니다.

- __*Buffers*__ 는 파이프라인의 element들 사이에서 스트리밍 데이터를 전달하기 위한 객체입니다. buffer들은 _source_ 에서 _sink_ 로 전달됩니다.

- __*Events*__ 는 element들 사이에서 혹은 application에서 element로 전달되는 객체입니다. event들은 upstream에서 downstream으로 전달될 수 있습니다. downstream event들은 데이터 흐름과 동기화 될 수 있습니다.

- __*messages*__ 는 파이프라인의 message bus에 있는 element들에 의해 기록되는 객체입니다. message들은 application에서 수집하기 위해 저장됩니다. message들은 message를 기록하는 element의 스트리밍 스레드로부터 동기적으로 중간에 붙들릴 수 있지만, message들은 보통 application의 메인 스레드에 의해 비동기적으로 처리됩니다. message들은 에러, 태그, 상태변화, 버퍼링상태, redirect 같은 정보들을 element로부터 application으로 thread-safe 방식으로 전송하는데에 사용됩니다.. 

- __*queries*__ 는 application들이 파이프라인으로부터 현재 재생 지점이나 지속시간과 같은 정보를 요청할 수 있도록 해줍니다. query들은 항상 동기적으로 응답합니다. element들은 그들과 같은 계층의 element들( 가령, 파일크기 혹은 지속시간 )로부터 정보를 요청할 수 있습니다. query들은 파이프라인 내에서 두 가지 방법으로 사용될 수 있지만, upstream에서의 query가 더 일반적입니다.

![communication](https://bleetoteelb.github.io/assets/img/communication.png)

.

.

.

.

.

.

.

.

.

.

.

.

.

.







































k