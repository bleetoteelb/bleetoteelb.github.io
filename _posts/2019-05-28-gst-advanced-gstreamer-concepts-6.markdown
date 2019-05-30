---
layout: post
title:  "Advanced GStreamer concepts - Dynamic Controllable Parameters"
subtitle:   "Dynamic Controllable Parameters"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>
<span class="fill_color"></span>

# __Dynamic Controllable Parameters__

## __1. Getting Started__

&nbsp; Gstreamer 속성들은 보통 <span class="fill_color">gst_object_set()</span>을 이용해 설정되지만, 속성 변경이 특정 스트림 시간에 영향을 끼치지 않도록 이러한 호출 타이밍을 안정적으로 잡는것은 불가능에 가깝습니다. 제어 시스템은 스트림 시간에 영향을 주지 않고 __GObject__ 속성을 조정하는 간단한 방법을 제공합니다.

&nbsp; 제어부에서는 시간을 고려하기에 control-binding을 이용하여 __GstControlSource__ 를 속성에 붙임으로써 동작하게 합니다. control-source는 0.0~1.0 사이로 주어지는 time-stamp 값을 제공합니다. control-binding은 control-value를 그들이 결합된 __GObject__ 속성에 연결하여 타입과 배율을 대상 속성의 값 범위로 변환합니다. 실행 중에 element는 현재 스트림 시간으로 __GObject__ 속성을 갱신하기 위해 지속적으로 값 변화를 감지합니다. Gstreamer는 이미 몇몇 다른 __GstControlSource__ 와 control-binding을 포함하고 있지만, application은 각각의 기본 클래스를 하위 클래스로 분류하여 자체정의 할 수 있습니다.

&nbsp; 대부분의 제어부 체계에는 __GstObject__ 가 적용되어 있습니다. __GstControlSource__ 와 control-binding을 위한 기본 클래스들은 중심 라이브러리에 포함되어 있지만, 기존에 구현된 것들은 __gstcontroller__ 에 포함되어 있으므로 필요에 따라 application의 소스 파일에 이러한 헤더들을 포함해야합니다.

```c
#include <gst/gst.h>
#include <gst/controller/gstinterpolationcontrolsource.h>
#include <gst/controller/gstdirectcontrolbinding.h>
...
```

&nbsp; 적절한 헤더들을 포함한 이후에, application은  __gstreamer-controller__ shared library에 연결해야 합니다. 요구되는 컴파일러를 얻고 연결 플래그를 얻기 위해서 다음을 사용해야 합니다.
```bash
pkg-config --libs --cflags gstreamer-controller-1.0
```

## __2. Setting up parameter control__

&nbsp; 만약 파이프라인이 설정되고 몇몇 인자들을 조정하고 싶다면, 먼저 __GstControlSource__ 를 생성해야합니다. interpolation __GstControlSource__ 를 사용해 봅시다.
```c
csource = gst_interpolation_control_source_new ();
g_object_set (csource, "mode", GST_INTERPOLATION_MODE_LINEAR, NULL);
```

&nbsp; 이제 우리는 __GstControlSource__ 를 __GObject__ 속성에 붙여봅시다. 이는 control-binding에 의해 수행됩니다. 별개의 control-bindings를 이용하여 한개의 control source는 여러 객체 속성에 붙을 수 있습니다. (심지어 다른 객체에도!)

```c
gst_object_add_control_binding (object, gst_direct_control_binding_new (object, "prop1", csource));
```

&nbsp; 이 타입의 __GstControlSource__ 는 새로운 속성값을 time-stamped 인자 변화 목록에서 가져옵니다.