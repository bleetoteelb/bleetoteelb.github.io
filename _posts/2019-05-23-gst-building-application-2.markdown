---
layout: post
title:  "Building an Application(2)"
subtitle:   "Elements"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Elements__

&nbsp; application 개발자에게 Gstreamer에서 가장 중요한 객체는 <span class="fill_color">GstElement</span>입니다. element는 미디어 파이프라인의 기본 구성 요소입니다. 모든 상위 수준에서의 구성요소들은 <span class="fill_color">GstElement</span>에서 얻을 수 있습니다. 모든 디코더, 인코더, 디먹서, 비디오 및 오디오 산출물은 사실 모두 <span class="fill_color">GstElement</span>입니다.

## __1. Wath are elements?__

&nbsp; application 개발자에게 element들은 블랙 박스로 가장 잘 시각화됩니다. 한쪽 끝에서 무언가를 넣을 수 있고, element는 그것을 이용해 무언가를 해서 다른쪽으로 내보냅니다. 예를들어 디코더 element에서 인코딩 된 데이터를 한쪽으로 넣으면 element는 디코딩 된 데이터를 다른 쪽으로 내보냅니다. 다음 장에서는 ([pad and capabilites](https://bleetoteelb.github.io/devlog/2019/05/22/gst-building-application-5/) 부분) 들어왔다 나가는 데이터들에 대해서와 application 안에서 어떻게 데이터들이 설정되는지를 설명합니다.

### __1.1 Source elements__

&nbsp; Source element들은 파이프라인에서 사용되는 데이터를 생성합니다. 예를들어 디스크 혹은 사운드 카드에서 정보를 읽는 것과 같은 일들을 이 부분에서 담당합니다. 다음 그림은 source element를 시각화 한 것인데 source pad는 항상 element의 오른쪽에 위치합니다.  
![src-element](https://bleetoteelb.github.io/assets/img/src-element.png)

&nbsp; Source element들은 데이터를 받지 않고 오직 생성만 합니다. source element는 오직 source pad만을 가지고 있는 것을 위 그림에서 볼 수 있습니다. source pad는 오직 데이터를 생성합니다.

### __1.2 Filters, convertors, demuxers, muxers and codecs__

&nbsp; filter들과 유사 filter element들은 입/출 pad를 모두 갖고 있습니다. 이 element에서는 입력(sink) pad에서 데이터를 받고, 출력(source) pad에 데이터를 제공합니다. 이러한 element들로는 volume element(filter), video scaler(convertor), Ogg debuxer 혹은 vorbis decoder 등이 있습니다.

&nbsp; 유사 filter element들은 여러개의 source 혹은 sink pad를 가질 수 있습니다. 예를들어 video demuxer는 한개의 sink pad와 여러개의 source pad를 갖고, 컨테이너 형식으로 각각의 기본 스트림에 포함됩니다. 반면에 디코더는 한개의 source pad와 sink pad를 갖고 있습니다.  
![filter-element.png](https://bleetoteelb.github.io/assets/img/filter-element.png)

&nbsp; 위 그림에서는 유사 filter element가 도식화 되어 있는데, 이 특별한 element는 한개의 source pad와 한개의 sink pad를 갖고 있습니다. sink pad는 데이터를 입력받는 역할을 하는데 그림에서 항상 왼쪽에 그려질 것입니다.

![filter-element-multi.png](https://bleetoteelb.github.io/assets/img/filter-element-multi.png)

위 그림에서는 또다른 유사 filter element가 도식화 되어 있는데, 여기서는 한개의 element안에 여러개의 출력(source) pad를 가질 수 있음을 보여줍니다. 이러한 element들의 예로는 오디오와 비디오를 모두 포함하는 Ogg 스트림을 처리하는 Ogg demuxer가 있습니다. Demuxer는 보통 새로운 pad가 생성될 때 시그널을 내보냅니다. 그러므로 application 프로그래머는 새로운 기본 스트림을 신호처리기에서 처리해야 합니다.

<br>

### __1.3 Sink elements__

&nbsp; Sink element들은 미디어 파이프라인에서 사용자(end-point)입니다. 그들은 데이터를 받지만 어떠한 것도 출력하지 않습니다. 디스크 쓰기, 사운드카드 재생, 비디오 재생 등이 이 sink element들에 의해 이루어집니다. sink element는 다음과 같이 도식화 할 수 있습니다.
![sink-element.png](https://bleetoteelb.github.io/assets/img/sink-element.png)


### __2. Creating a GstElement__

&nbsp; 이 element를 만드는 가장 쉬운 방법은 <span class="fill_color">gst_element_factory_make()</span> 를 이용하는 것입니다. 이 함수는 팩토리 이름을과 element 이름을 받아서 새로운 element를 생성합니다.
![linked-elements.png](https://bleetoteelb.github.io/assets/img/linked-elements.png)