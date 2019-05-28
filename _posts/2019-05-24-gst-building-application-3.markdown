---
layout: post
title:  "Building an Application - Bins"
subtitle:   "Bins"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Bins__

&nbsp; bin은 element 컨테이너입니다. element를 bin에 추가할 수도 있지만, bin이 element 그 자체가 될 수 있기 때문에, bin은 다른 element와 같은 방법으로 다뤄질 수 있다. 그러므로 이전 파트(Elements)의 모든 내용이 bin에도 적용됩니다.

## __1. What are bins__

&nbsp; bin은 링크된 element를 한개의 논리적 element의 그룹으로 묶을 수 있게 해줍니다. 개별적으로 element들을 더 이상 처리하지 않고 묶어서 한개의 element (혹은 bin)을 처리하면 됩니다. 이것은 당신이 복잡한 파이프라인을 구성할 때 파이프라인을 더 작은 조각으로 쪼갤 수 있도록 해주기 때문에 매우 강력한 수단입니다.

&nbsp; bin은 또한 안에 담겨있는 element들을 관리합니다. bus 메세지를 전달하고 모을 뿐 아니라 element의 상태 변화를 수행하기도 합니다.

![bin-element](https://bleetoteelb.github.io/assets/img/bin-element.png)

&nbsp; Gstreamer 개발자가 사용가능한 특별한 타입의 bin이 한가지 있습니다.
- 파이프라인 : 포함된 element의 동기화 및 bus 메세지들을 관리하는 일반적인 컨테이너. 최상위 bin은 파이프라인이어야 하며, 모든 application은 최소한 이것들 중 하나를 필요로 합니다.

## __2. Creating a bin__

&nbsp; bin은 element가 생성되는 것과 같은 방식으로 생성됩니다. bin과 관련해서도 편리한 함수들이 이미 존재하는데, bin생성을 위해서는 <span class="fill_color">gst_bin_new()</span>와 <span class="fill_color">gst_pipeline_new()</span>를 사용하고, element를 bin에 추가하거나 bin에서 삭제하고 싶을 때는 <span class="fill_color">gst_bin_Add()</span>과 <span class="fill_color">gst_bin_remove()</span>를 사용하면 됩니다. element를 추가하는 bin은 그 element의 소유권을 가져간다는 사실을 기억해야 합니다. 만약 bin을 파괴한다면, 그 안에 있는 element들은 그 bin에 대한 참조를 해제할 것입니다. 만약 element를 bin에서 제거한다면, 자동적으로 참조가 해제될것입니다.

```c
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElement *bin, *pipeline, *source, *sink;

  /* init */
  gst_init (&argc, &argv);

  /* create */
  pipeline = gst_pipeline_new ("my_pipeline");
  bin = gst_bin_new ("my_bin");
  source = gst_element_factory_make ("fakesrc", "source");
  sink = gst_element_factory_make ("fakesink", "sink");

  /* First add the elements to the bin */
  gst_bin_add_many (GST_BIN (bin), source, sink, NULL);
  /* add the bin to the pipeline */
  gst_bin_add (GST_BIN (pipeline), bin);

  /* link the elements */
  gst_element_link (source, sink);

[..]

}
```

&nbsp; 위 코드는 물론 말도안되는 예제코드이다. playbin element와 같이 더 강력하고 유연한 custom bin이 있다.

&nbsp; custom bin들은 plugin과 함께 생성되거나 application으로부터 생성됩니다. cutom bin을 생성하는 것에 대한 자세한 정보는 [Plugin Writer's Guide](https://gstreamer.freedesktop.org/documentation/plugin-development/index.html?gi-language=c)에서 찾으십시오.

&nbsp; Custom bin의 예시는 [gst-plugins-base](https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gst-plugins-base-plugins/html/index.html)에 있는 playbin과 uridecodebin element입니다.

## __3. Bins manage states of their children__

&nbsp; bin들은 속해있는 모든 element의 상태를 관리합니다. 만약 bin(혹은 bin의 최상위 레벨 타입인 파이프라인)을 <span class="fill_color">gst_element_set_state()</span>를 사용하여 특정한 상태로 설정하면, 그 안에 있는 모든 element들은 그 상태로 변경될 것입니다. 이는 파이프라인을 시작하거나 종료할 때 최상위 레벨의 파이프라인의 상태만 설정해주면 된다는 것을 의미합니다.

&nbsp; bin은 sink element부터 source element까지 하위의 모든 element의 상태 변화를 수행합니다. 이것은 upstream element가 PAUSED 혹은 PLAYING일 때, downstream element가 데이터를 받을 준비가 되었다는 것을 의미합니다. 마찬가지로 파이프라인이 종료될 때, sink element가 READY 혹은 NULL상태가 먼저 되고, upstream element가 FLUSHING 에러를 받고 element들이 READY 혹은 NULL 상태로 변하기 전에 스트리밍을 멈춥니다.

&nbsp; 그러나 element들이 이미 동작중인 bin이나 파이프라인에 추가된다면(예를 들면 "padd-added" 신호 콜백 중에), 그것이 추가된 bin의 상태와 자동으로 동기화되지 않습니다. 때문에 이미 동작중인 파이프라인에 element를 추가할  때에는 <span class="fill_color">gst_element_set_state()</span>혹은 <span class="fill_color">gst_element_sync_state_with_parent()</span>함수를 이용하여 따로 상태를 설정해주어야 합니다.