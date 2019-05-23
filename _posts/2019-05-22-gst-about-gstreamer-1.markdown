---
layout: post
title:  "About Gstreamer(1)"
subtitle:   "Gstreamer란"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Gstreamer란?__
 
 &nbsp; Gstreamer는 스트리밍 영상 재생 프로그램을 제작하기 위한 프레임워크입니다. 기본적인 설계는 _DirectShow_ 의 아이디어와 _Oregon Graduate Institute_ 의 비디오 파이프라인에 기반합니다.

 &nbsp; Gstreamer의 개발 프레임워크를 사용하면 모든 종류의 스트리밍 멀티미디어 프로그램을 작성할 수 있습니다. Gstreamer 프레임워크는 비디오나 오디오(혹은 둘다) 다루는 프로그램을 만들기 쉽도록 설계되어 있습니다. 사실, 오디오 및 비디오에만 국한되지 않고 모든 종류의 데이터 흐름을 처리 할 수 있습니다. 파이프라인 설계는 적용된 필터가 야기하는 것보다 적은 오버헤드를 갖도록 만들어졌습니다. 그러므로 Gstreamer는 긴 대기시간이 요구되는 고급 audio 프로그램에게도 좋은 프레임워크입니다.

 &nbsp; Gstreamer의 가장 명백한 용도 중 하나는 이를 사용하여 미디어 플레이어를 제작하는 것입니다. Gstreamer에는 이미 _Mp3, Ogg/Vorbis, MPEG-1/2, AVI, Quicktime, mod_ 등을 포함한 매우 다양한 형식을 지원할 수 있는 미디어 플레이어 제작을 위한 요소들을 포함하고 있습니다. 그러나 Gstreamer는 다른 미디어 플레이어 그 이상입니다. Gstreamer의 주요 장점은 확장 가능한 구성요소들이 혼합되어 임이의 파이프라인에 연결될 수 있어 온전한 비디오/오디오 편집 프로그램을 제작할 수 있다는 점입니다.

 &nbsp; 이 프레임워크는 다양한 코덱과 기타 기능들을 제공하는 플러그인을 기반으로 작동합니다. 플러그인들은 파이프라인에서 연결되거나 정렬될 수 있는데, 이 파이프라인은 데이터의 흐름을 정의합니다. 파이프라인들은 GUI 편집기에 의해 편집되기도 하고 XML의 형태로 저장되어 파이프라인 라이브러리들이 최소한으로 만들어지도록 합니다.

 &nbsp; Gstreamer의 핵심 기능은 플러그인들, 데이터 흐름과 미디어 타입의 처리를 위한 프레임워크를 제공하는 것입니다. 또한 Gstreamer는 다양한 플러그인을 사용하여 application의 제작을 위한 API를 제공합니다.

 &nbsp; 다음은 Gstreamer가 제공하는 기능들입니다.

- an API for multimedia applications
- a plugin architecture
- a pipeline architecture
- a mechanism for media type handling/negotiation
- a mechanism for synchronization
- over 250 plug-ins providing more than 1000 elements
- a set of tools

<br>

Gstreamer의 플러그인들은 다음과 같이 분류될 수 있습니다.

- protocols handling
- sources: for audio and video (involves protocol plugins)
- formats: parsers, formaters, muxers, demuxers, metadata, subtitles
- codecs: coders and decoders
- filters: converters, mixers, effects, ...
- sinks: for audio and video (involves protocol plugins)

![gstreamer-overview](https://bleetoteelb.github.io/assets/img/gstreamer-overview.png)  

<br>

&nbsp; Gstreamer는 다음과 같이 묶여 있습니다.

- gstreamer: the core package
- gst-plugins-base: an essential exemplary set of elements
- gst-plugins-good: a set of good-quality plug-ins under LGPL
- gst-plugins-ugly: a set of good-quality plug-ins that might pose distribution problems
- gst-plugins-bad: a set of plug-ins that need more quality
- gst-libav: a set of plug-ins that wrap libav for decoding and encoding
- a few others packages

