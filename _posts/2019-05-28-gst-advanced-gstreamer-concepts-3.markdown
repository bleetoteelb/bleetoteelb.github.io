---
layout: post
title:  "Advanced GStreamer concepts - Interfaces"
subtitle:   "Interfaces"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>
<span class="fill_color"></span>

# __Interfaces__

&nbsp; [Using an element as a GObject](https://gstreamer.freedesktop.org/documentation/application-development/basics/elements.html?gi-language=c#using-an-element-as-a-gobject) 에서는 상호 작용 할 수 있는 application과 element가 상호작용할 수 있는 간단한 방법으로 __GObject__ 속성을 사용하고 있습니다. 이 방법은 단순한 getter 혹은 setter에 대해서는 충분하지만 더 복잡한 경우에는 문제가 생길 수 있습니다. 보다 복잡한 코드에서 Gstreamer는 __GObject__ __GTypeInterface__ 타입을 기반으로 인터페이스를 사용합니다.

&nbsp; 이 글은 입문용이며 예제 코드는 포함하지 않습니다. 더 자세한 내용들은 API reference를 확인하십시오.

## __1. The URI Handler interface__

&nbsp; 지금까지의 예제에서 우리는 "filesrc" element를 사용하여 로컬 파일 읽기를 보여주었는데, Gstreamer는 다른 방식으로도 source를 받을 수 있습니다.

&nbsp; Gstreamer는 특정 네트워크 소스 타입에 사용할 element와 같은 URI 특성을 application에게 요구하지 않습니다. 이러한 세부사항들은 __GstURIHandler__ 인터페이스에 의해 수행됩니다.

&nbsp; URI 명명법에는 엄격한 규칙은 없지만, 보편적으로 쓰이는 명명 규칙을 따릅니다. 예를들어 plugin이 제대로 설치되어있다는 전제하에 Gstreamer는 다음과 같은 형식을 지원합니다.
```bash
file:///<path>/<file>
http://<host>/<path>/<file>
mms://<host>/<path>/<file>
dvb://<CHANNEL>
...
```

&nbsp; 특정 URI를 지원하는 source 혹은 sink elemenet를 얻기 위해서 필요한 방향에 따라 __GST_URI_SRC__ 또는 __GST_URI_SINK__ 를 사용하는 <span class="fill_color">gst_element_make_from_uri()</span>를 사용해야 합니다.

&nbsp; GLib에 있는 <span class="fill_color">g_filename_to_uri()</span> 혹은 <span class="fill_color">g_uri_to_filename()</span>을 이용하여 __filename->URI / URI->filename__ 변환을 할 수도 있습니다.

## __2. The Color Balance interface__

&nbsp; __GstColorBalance__ 인터페이스는 밝기,대조와 같은 비디오와 관련된 element의 속성들을 제어하기 위한 방법입니다. 그 존재의 유일한 이유는 __GObject__ 를 사용하여 동적으로 속성을 등록할 수 있는 방법이 없기 때문입니다. 

&nbsp; colorbalance 인터페이스는 _xvimagesink_, _glimagesink_ 및 _Video4linux2_ element를 포함하는 여러 플러그인들에 의해 구현됩니다.

## __3. The Video Overlay interface__

&nbsp; __GstVideoOverlay 인터페이스는 application 창에 비디오 스트림을 내장하는 문제를 해결하기 위해 만들어졌습니다. application은 이 인터페이스를 적용하는 element를 다루는 창을 제공하고, element는 이 창을 이용하여 새로운 상위 레벨의 창을 만들지 않고 이 문제를 처리할 것입니다.

&nbsp; 이 인터페이스는 _Video4linux2_ element와 _slightagesink_, _ximagesink_, _xvimagesink_ 및 _sdlvideosink_에 의해 구현됩니다.

## __4. Other interfaces__

&nbsp; element에 의해 적용되고 Gstreamer에 의해 제공되는 몇가지 다른 인터페이스들이 있습니다.

- <span class="fill_color">GstChildProxy</span> For access to internal element's properties on multi-child elements
- <span class="fill_color">GstNavigation</span> For the sending and parsing of navigation events
- <span class="fill_color">GstPreset</span> For handling element presets
- <span class="fill_color">GstRTSPExtension</span> An RTSP Extension interface
- <span class="fill_color">GstStreamVolume</span> Interface to provide access and control stream volume levels
- <span class="fill_color">GstTagSetter</span> For handling media metadata
- <span class="fill_color">GstTagXmpWriter</span> For elements that provide XMP serialization
- <span class="fill_color">GstTocSetter</span> For setting and retrieval of TOC-like data
- <span class="fill_color">GstTuner</span> For elements providing RF tunning operations
- <span class="fill_color">GstVideoDirection</span> For video rotation and flipping controls
- <span class="fill_color">GstVideoOrientation</span> For video orientation controls
- <span class="fill_color">GstWaylandVideo</span> Wayland video interface