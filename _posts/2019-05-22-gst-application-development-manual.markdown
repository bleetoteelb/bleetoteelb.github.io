---
layout: post
title:  "Application Development Manual"
subtitle:   "Application Development Manual"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __0. 개요__

&nbsp; 이 문서는 Gstreamer를 처음 사용하는 사람들을 위해 제공되는 document를 번역하여 좀 더 빠르고 쉽게 접근하게 하기 위한 목적으로 작성하게 된 문서입니다.

&nbsp; 원본글을 구글번역기를 돌리면 그만이긴 하겠지만 ~~(요새 구글번역기 수준이 굉장히 높더라..)~~, 좀 더 매끄러운 번역으로 정리하여 도움을 주고자 한다. 일부 설명 구조를 각색하거나 변형한 경우도 있습니다. (물론 내용을 빼지는 않았다.)

&nbsp; 원본은 [Application Development Manual](https://gstreamer.freedesktop.org/documentation/application-development/index.html?gi-language=c) 에서 확인할 수 있습니다.

<span style="color:red"> 참고) 작성일은 __2019.05.22__ 이며 이후에 업데이트된 사이트의 내용은 반영되지 않았을 수 있습니다.</span>

<br>

 ## __목차__


### __Application Development Manual__
- About Gstreamer(1) : [What is Gstreamer?](https://bleetoteelb.github.io/devlog/2019/05/22/gst-about-gstreamer-1/)
- About Gstreamer(2) : [Design principles](https://bleetoteelb.github.io/devlog/2019/05/22/gst-about-gstreamer-2/)
- About Gstreamer(3) : [Foundations](https://bleetoteelb.github.io/devlog/2019/05/22/gst-about-gstreamer-3/)  
&nbsp;
- Building an Application(1) : [Initializing Gstreamer](https://bleetoteelb.github.io/devlog/2019/05/22/gst-building-application-1/) 
- Building an Application(2) : [Elements](https://bleetoteelb.github.io/devlog/2019/05/22/gst-building-application-2/) 
- Building an Application(3) : [Bins](https://bleetoteelb.github.io/devlog/2019/05/22/gst-building-application-3/) 
- Building an Application(4) : [Bus](https://bleetoteelb.github.io/devlog/2019/05/22/gst-building-application-4/) 
- Building an Application(5) : Pads and capabilities
- Building an Application(6) : Buffers and Events
- Building an Application(7) : Your first application  
&nbsp;
- Advanced Gstreamer concepts(1) : Position tracking and seeking
- Advanced Gstreamer concepts(2) : Metadata
- Advanced Gstreamer concepts(3) : Interfaces
- Advanced Gstreamer concepts(4) : Clocks and synchronization in Gstreamer
- Advanced Gstreamer concepts(5) : Buffering
- Advanced Gstreamer concepts(6) : Dynamic Controllable Parameters
- Advanced Gstreamer concepts(7) : Threads
- Advanced Gstreamer concepts(8) : Autoplugging
- Advanced Gstreamer concepts(9) : Pipeline manipulation  
&nbsp;
- Higher-level interfaces for Gstreamer applications : Playback Components  
&nbsp;
- Appendices(1) : Programs
- Appendices(2) : Compiling
- Appendices(3) : Things to check when writing an application
- Appendices(4) : Porting 0.8 applications to 0.10
- Appendices(5) : Porting 0.10 applications to 1.0
- Appendices(6) : Integration
- Appendices(7) : Licensing advisory
- Appendices(8) : Quotes from the Developers


<br>

# __1. Document에 앞서..__

&nbsp; Gstreamer는 스트리밍 미디어 application을 만들기 위한 매우 강력하고 다목적의 프레임 워크입니다. Gstreamer의 장점의 상당수는 Gstreamer의 모듈성에 기반합니다: Gstreamer는 매끄럽게 새로운 플러그인 모듈들을 연결해 줄 수 있습니다. 그러나 모듈성으로 인해 복잡도가 발생하기 때문에, 새로운 application을 만드는 것이 쉬운 것만은 아닙니다.

&nbsp; 이 문서는 당신이 Gstreamer framework를 이해하여 이를 바탕으로 application을 개발하는데에 도움을 주고자 합니다. 첫 번째 장에서는 Gstreamer 개념을 이해하는 데 도움이 되는 간단한 오디오 플레이어를 만들어 볼 것입니다. 그 이후 장에서는 미디어 재생 및 기타 미디어 처리(캡처,편집 등)와 관련된 고급 주제에 대해 설명할 것입니다.

<br>

# __2. Introduction__

## __2.1. 이 메뉴얼을 읽어야할 대상__

&nbsp; 이 문서는 application 개발자의 관점에서 Gstreamer를 설명합니다. 이 문서에서는 Gstreamer 라이브러리와 툴을 사용하여 Gstreamer application을 어떻게 작성하는지 설명합니다. 플러그인들을 작성하는 방법에 대한 설명은 [Plugin Writer's Guide](https://gstreamer.freedesktop.org/documentation/plugin-development/index.html?gi-language=c)를 참조하세요.

&nbsp; 그 밖에 Gstreamer에 관련된 모든 문서는 [Gstreamer web site](https://gstreamer.freedesktop.org/documentation/doc_index.html?gi-language=c)에 모두 있으니 필요한 문서는 참고하시기 바랍니다.

<br>


## __2.2. 선행 지식__

&nbsp; 이 메뉴얼을 이해하기 위해 C언어에 대한 기본적인 이해가 있어야 한다. Gstreamer는 GObject 프로그래밍 모델에 기반하고 있기 때문에, 이 문서 역시 GObject와 glib에 대해 기본적으로 알고있다고 가정할 것이다.

특히 다음 내용은 반드시 이해하고 있어야 한다.

- GObject instantiation

- GObject properties (set/get)

- GObject casting

- GObject referencing/dereferencing

- glib memory management

- glib signals and callbacks

- glib main loop

<br>

## __2.3. 메뉴얼 구조__

&nbsp; 이 가이드는 몇가지 큰 파트로 나뉩니다. 각 파트는 GStreamer 어플리케이션 개발과 관련된 특정한 주제들을 다룹니다. 이 가이드의 파타들은 다음 순서로 구성되어있습니다.

1. <span class="fill_color">About Gstreamer</span> 에서는 Gstreamer의 개요,디자인 원리와 기초를 설명합니다.

2. <span class="fill_color">Building and Application</span> 에서는 Gstreamer application의 기본 사항을 다룹니다. 이 파트가 끝나면 당신은 Gstreamer를 이용해서 audio player를 만들 수 있어야 합니다.

3. <span class="fill_color">Advanced Gstreamer concepts</span> 에서는 Gstreamer를 경쟁 프로그램보다 돋보이는 부분에 대해 다룹니다. 우리는 dynamic parameters와 interfaces를 이용하여 프로그램-파이프라인의 상호작용에 대해 이야기 할 것이며, 스레드, 스레드 파이프라인, 스케쥴링과 동기화에 대한 내용도 다룰것입니다. 이 주제들 태부분은 단순히 API를 소개하는것 뿐 아니라, 주로 Gstreamer를 사용하여 프로그래밍 문제를 해결하고 개념을 이해하는 방법에 대해 보다 심층적인 통찰력을 제공하고자 합니다.

4. <span class="fill_color">Higher-level interfaces for GStreamer applications</span> 에서는 Gstreamer를 위한 더 높은 수준의 프로그래밍 API들에 대해 다룹니다. 이 부분을 이해하기 위해 이전의 파트들을 자세하게 알고있을 필요는 없습니다. 그러나 Gstreamer 기본 개념에 대해서는 이해하고 있어야 합니다. 이 파트에서 playbin과 autopluggers에 대해 다룰것입니다.

5. <span class="fill_color">Appendices</span> 에서는 GNOME,KDE,OS X 또는 Windows와의 통합, 디버깅 도움말 및 GStreamer 프로그래밍을 개선하고 단순화하기 위한 일반적인 팁에 대해 다룰 것입니다.