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

&nbsp; filter들과 filter-like element들은 입/출 pad를 모두 갖고 있습니다. 이 element에서는 입력(sink) pad에서 데이터를 받고, 출력(source) pad에 데이터를 제공합니다. 이러한 element들로는 volume element(filter), video scaler(convertor), Ogg debuxer 혹은 vorbis decoder 등이 있습니다.

&nbsp; filter-like element들은 여러개의 source 혹은 sink pad를 가질 수 있습니다. 예를들어 video demuxer는 한개의 sink pad와 여러개의 source pad를 갖고, 컨테이너 형식으로 각각의 기본 스트림에 포함됩니다. 반면에 디코더는 한개의 source pad와 sink pad를 갖고 있습니다.  
![filter-element.png](https://bleetoteelb.github.io/assets/img/filter-element.png)

&nbsp; 위 그림에서는 filter-like element가 도식화 되어 있는데, 이 특별한 element는 한개의 source pad와 한개의 sink pad를 갖고 있습니다. sink pad는 데이터를 입력받는 역할을 하는데 그림에서 항상 왼쪽에 그려질 것입니다.

![filter-element-multi.png](https://bleetoteelb.github.io/assets/img/filter-element-multi.png)

위 그림에서는 또다른 filter-like element가 도식화 되어 있는데, 여기서는 한개의 element안에 여러개의 출력(source) pad를 가질 수 있음을 보여줍니다. 이러한 element들의 예로는 오디오와 비디오를 모두 포함하는 Ogg 스트림을 처리하는 Ogg demuxer가 있습니다. Demuxer는 보통 새로운 pad가 생성될 때 시그널을 내보냅니다. 그러므로 application 프로그래머는 새로운 기본 스트림을 신호처리기에서 처리해야 합니다.

<br>

### __1.3 Sink elements__

&nbsp; Sink element들은 미디어 파이프라인에서 사용자(end-point)입니다. 그들은 데이터를 받지만 어떠한 것도 출력하지 않습니다. 디스크 쓰기, 사운드카드 재생, 비디오 재생 등이 이 sink element들에 의해 이루어집니다. sink element는 다음과 같이 도식화 할 수 있습니다.  
![sink-element.png](https://bleetoteelb.github.io/assets/img/sink-element.png)


## __2. Creating a GstElement__

&nbsp; 이 element를 만드는 가장 쉬운 방법은 <span class="fill_color">gst_element_factory_make()</span> 를 이용하는 것입니다. 이 함수는 팩토리 이름을과 element 이름을 받아서 새로운 element를 생성합니다. element의 이름은 나중에 bin 안의ㅏ element를 조회할 때 사용될 것입니다. 그 이름은 디버그 출력에 사용되기 때문에 고유한 기본 이름을 얻기 위해 null을 전달 할 수도 있습니다.

&nbsp; element가 더 이상 필요하지 않을 때, <span class="fill_color">gst_object_unref()</span>를 이용해서 unref해야 합니다. 이 함수는 element의 참조 카운트를 1 감소시킵니다. element의 참조카운트는 생성되었을 때 1이며, 참조카운트가 0이 될 때 완전하게 삭제됩니다.

&nbsp; 다음은 _fakesrc_라는 이름의 element factory안에 _source_ 라는 이름의 element를 어떻게 생성하는지를 보여주는 예제코드입니다. 여기서 생성이 완료됐음을 확인하고 생성된 element를 unref(참조 카운트를 1 감소하는 것)을 할 것입니다.

```c
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElement *element;

  /* init GStreamer */
  gst_init (&argc, &argv);

  /* create element */
  element = gst_element_factory_make ("fakesrc", "source");
  if (!element) {
    g_print ("Failed to create element of type 'fakesrc'\n");
    return -1;
  }

  gst_object_unref (GST_OBJECT (element));

  return 0;
}
```

&nbsp; <span class="fill_color">gst_element_factory_make</span>는 사실 두 함수의 조합입니다.(gst_element_factory_find & gst_element_factory_create) GstElement 객체는 factory에서 생성됩니다. element를 생성하기 위해서는 고유의 factory 이름을 사용하여 GstElementFactory 객체에 접근해야 합니다. 이 역할을 <span class="fill_color"> gst_element_factory_find()</span> 에서 담당합니다.


<br>

&nbsp; 다음 예제코드는 가짜 데이터를 공급하는 fakesrc element를 생성할 때 사용될 수 있는 factory를 얻기 위해 사용되는 코드입니다. <span class="fill_color"> gst_element_factory_create()</span> 함수는 주어진 이름으로 element를 생성하기위해 element factory 이름을 사용합니다.

```c
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElementFactory *factory;
  GstElement * element;

  /* init GStreamer */
  gst_init (&argc, &argv);

  /* create element, method #2 */
  factory = gst_element_factory_find ("fakesrc");
  if (!factory) {
    g_print ("Failed to find factory of type 'fakesrc'\n");
    return -1;
  }
  element = gst_element_factory_create (factory, "source");
  if (!element) {
    g_print ("Failed to create element, even though its factory exists!\n");
    return -1;
  }

  gst_object_unref (GST_OBJECT (element));
  gst_object_unref (GST_OBJECT (factory));

  return 0;
}
```

<br>

<br>

## __3. Using an element as a GObject__

&nbsp; __GstElement__ 표준 __GObject__ 속성을 사용하여 적용된 여러가지 속성들을 가질 수 있습니다. 그러므로 GObject가 query를 보내고, 속성값을 설정하거나 얻는 일반적인 방법들 역시 지원됩니다.

&nbsp; 모든 __GstElement__ 들은 부모 GstObject로부터 최소 하나 이상의 속성을 상속받습니다.(최소한인 그 한개는 "이름"속성입니다.) 이 "이름" 속성은<span class="fill_color"> gst_element_factory_create()</span> 함수와 <span class="fill_color"> gst_element_factory_find()</span>함수에 당신이 제공하는 이름입니다. 당신은 이 속성을 <span class="fill_color"> gst_element_set_name()</span>함수와 <span class="fill_color"> gst_element_get name()</span>함수를 이용해 설정하고 얻을 수 있고, 다음 예제코드와 같이 __GObject__ 속성 체계를 이용할 수 있습니다.

```c
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElement *element;
  gchar *name;

  /* init GStreamer */
  gst_init (&argc, &argv);

  /* create element */
  element = gst_element_factory_make ("fakesrc", "source");

  /* get name */
  g_object_get (G_OBJECT (element), "name", &name, NULL);
  g_print ("The name of the element is '%s'.\n", name);
  g_free (name);

  gst_object_unref (GST_OBJECT (element));

  return 0;
}
```

&nbsp; 대부분의 플러그인들은 element를 설정하기 위한 설정에 대한 정보들을 더 제공하기 위해 추가적인 속성들을 제공합니다. __gst-inspect__은 특정 element의 속성을 요청하는 데 유용한 도구이며, 속성의 역할과 그 인자타입 및 범위에 대한 간략한 설명을 제공하기 위한 적절한 introspection을 사용합니다. __gst-inspect__ 에 대한 자세한 내용은 appendix의 [gst-inspect](https://gstreamer.freedesktop.org/documentation/application-development/appendix/checklist-element.html?gi-language=c#gst-inspect) 문서를 참고하십시오.

&nbsp; GObject 속성들에 관한 자세한 내용들은 [GObject manual](https://developer.gnome.org/gobject/stable/rn01.html)와 [The Glib Object system](https://developer.gnome.org/gobject/stable/pt01.html)문서를 참고하길 권장합니다.

&nbsp; __GstElement__ 는 유연한 콜백 체계로 사용되는 다양한 GObject 신호(Signal)들을 제공합니다. 특정한 element들이 어떤 신호들을 지원하는지에 대해 보기 위해 여기서도 __gst-inspect__ 를 사용할 수 있습니다. 신호들과 속성들은 element들과 application들이 상호작용하는 가장 기본적인 방법입니다.

## __4. More about element factories__

&nbsp; 지금까지 __GstElementFactory__ 객체에 대해서는 element의 인스턴스를 생성하는 방법으로 간단하게 소개했다. 그러나 element factory들은 그보다 훨씬 더 복잡한 객체이다. element factory들은 Gstreamer 레지스트리에서 가져온 기본 타입이며, Gstreamer가 생성할 수 있는 모든 프러그인들과 element들을 설명합니다. 이것은 element factory들이 자동으로 element를 인스턴스시키고 (autoplugger들이 하는 것처럼), 사용가능한 element의 목록을 생성하는 데에 유용하다는 것을 의미합니다.

### __4.1. Getting information about an element using a factory__

&nbsp;__gst-inspect__ 와 같은 도구들은 element(플러그인을 제작한 사람), 설명적 이름(및 짧은 이름),순위와 분류 와 같은 몇가지 일반적인 정보를 제공합니다. 여기서 분류는 그 element factory를 사용하여 생성된 element의 타입을 얻기 위해 사용됩니다. 예를들어 카테고리에는 __Codex/Decoder/Video__ (video decoder), __Codec/Encoder/Video__ (video encoder), __Source/Video__ (a video generator), __Sink/Video__ (a video output) 등이 있으며, 이러한 모든 분류들은 당연히 audio에 대해서도 같은 분류들이 있습니다. 그러므로 __Codec/Demuxer__ 및 __Codec/Muxer__ 를 포함해 더 다양한 분류들이 많습니다. __gst-inspect__ 는 모든 factory들의 목록을 제공하고 <span class="fill_color">gst-inspect \<factory-name></span> 은 위의 모든 정보들을 포함하여 많은 정보들을 나열합니다.

```c
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElementFactory *factory;

  /* init GStreamer */
  gst_init (&argc, &argv);

  /* get factory */
  factory = gst_element_factory_find ("fakesrc");
  if (!factory) {
    g_print ("You don't have the 'fakesrc' element installed!\n");
    return -1;
  }

  /* display information */
  g_print ("The '%s' element is a member of the category %s.\n"
           "Description: %s\n",
           gst_plugin_feature_get_name (GST_PLUGIN_FEATURE (factory)),
           gst_element_factory_get_metadata (factory, GST_ELEMENT_METADATA_KLASS),
           gst_element_factory_get_metadata (factory, GST_ELEMENT_METADATA_DESCRIPTION));

  gst_object_unref (GST_OBJECT (factory));

  return 0;
}
```
&nbsp; 위 코드에서 <span class="fill_color">gst_registry_pool_feature_list (GST_TYPE_ELEMENT_FACTORY)</span> 함수는 Gstreamer가 갖고 있는 모든 element factory들의 목록을 얻기 위해 사용되었습니다.


### __4.2. Finding out what pads an element can contain__

&nbsp; 아마 element factory들의 가장 강력한 특징은 그들이 실제로 플러그인을 메모리에 저장할 필요 없이, element가 생성한 pad와 그 pad의 기능(쉽게 말해, 어떤 타입의 media가 그 pad를 통해 스트리밍 될 수 있는지)에 대한 모든 정보를 포함하고 있다는 것입니다. 이것은 encoder에게 코덱 선택 목록을 제공하기 위해 사용될 수 있고, 미디어 플레이어를 위한 autoplugging 목적으로 사용될 수 있습니다. 모든 현재 Gstreamer에 기반한 미디어 플레이어와 autoplugger들은 이런 방식으로 작업합니다. __GstPad__ 와 __GstCaps__ 에 관련된 더 자세한 특징들은 이후에 나올 챕터인 [Pads and capabilities](https://gstreamer.freedesktop.org/documentation/application-development/basics/pads.html?gi-language=c)에서 다룰것입니다.

## __5. Linking elements__

&nbsp; source element와 0개 혹은 그 이상의 filter-like elements와, 마지막에넌 sink element와 링크함으로써, 미디어 파이프라인을 구성할 수 있습니다. 이 때, 데이터는 element를 통해 흐르게 되는데, 이 것은 Gstreamer에서 미디어 처리의 기본 개념입니다.

![linked-elements.png](https://bleetoteelb.github.io/assets/img/linked-elements.png)

&nbsp; 위 그림에서처럼 3개의 element들을 링크 함으로써, 우리는 매우 간단한 element 체인을 만들 수 있습니다. source element("element1")의 출력이 filter-like element("element2")의 입력으로 사용됩니다. filter-like element는 입력된 데이터로 무언가를 하고 그 결과를 마지막 sink element("element3")으로 전달합니다.

&nbsp; 위 그림을 간단한 _Ogg/Vorbis audio decoder_ 라고 생각해봅시다. 그러면 여기서 source는 이스크로부터 파일을 읽어들이는 disk source입니다. 두번째 element는 Ogg/Vorbis audio decoder이며 sink element는 디코딩된 오디오 데이터를 재생하는 사운드카드입니다. 우리는 이 문서에서 나중에 이 간단한 그래프를 Ogg/Vorbis 플레이어 구성을 위해 사용할 것입니다.

&nbsp; 위 그래프는 다음과 같이 코드로 작성할 수 있습니다.

```c
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  GstElement *pipeline;
  GstElement *source, *filter, *sink;

  /* init */
  gst_init (&argc, &argv);

  /* create pipeline */
  pipeline = gst_pipeline_new ("my-pipeline");

  /* create elements */
  source = gst_element_factory_make ("fakesrc", "source");
  filter = gst_element_factory_make ("identity", "filter");
  sink = gst_element_factory_make ("fakesink", "sink");

  /* must add elements to pipeline before linking them */
  gst_bin_add_many (GST_BIN (pipeline), source, filter, sink, NULL);

  /* link */
  if (!gst_element_link_many (source, filter, sink, NULL)) {
    g_warning ("Failed to link elements!");
  }

[..]

}
```

&nbsp; 보다 구체적인 동작을 위해 <span class="fill_color">gst_element_link ()</span>와 <span class="fill_color">gst_element_link_pads ()</span>함수도 있습니다. 개별 pad에 대한 reference를 얻고 다양한 <span class="fill_color">gst_pad_link_* ()</span> 함수를 사용하여 pad를 연결할 수도 있습니다. 자세한 내용은 API refernces를 참고하세요.

&nbsp; __중요__ : bin에 element를 추가하면  기존의 링크들이 모두 끊기기 때문에 element를 링크하기 전에 bin과 파이프라인에 element들을 추가해주어야 합니다. 또한 같은 bin 혹은 파이프라인에는 element를 직접적으로 연결할 수 없습니다. 계층구조의 다른 레벨에 element나 pad를 링크하고 싶다면, ghost pad를 사용해야 합니다. (ghost pad는 나중에 설명되어 있습니다.)

## __5. Element States__

&nbsp; 생성된 후에 element는 사실 아직 아무 동작도 하지 않을 것입니다. 어떤 동작을 하기 위해서 element의 상태를 바꾸어 주어야 하는데, Gstreamer에는 각각이 아주 특별한 의미를 가지고 있는 4가지 element 상태가 있습니다.

1. __GST_STATE_NULL__ : 기본 상태입니다. 이 상태에서는 할당 된 리소스가 없으므로 리소스로 전환하면 모든 리소스가 해제됩니다. 참조 카운트가 0이 되고 해제된 element들은 반드시 이 상태이어야 합니다.

2. __GST_STATE_READY__ : 이 상태에서 element는 글로벌 리소스, 즉 스트림 내에서 유지할 수 있는 리소스를 할당했습니다. 여기서 장치를 실행하거나 버퍼를 할당하는 것에 대해 생각해 볼 수 있지만, 스트림은 이 상태에서는 열리지 않으므로 스트림의 위치는 자동으로 0입니다. 만약 스트림이 이전에 읽혔다면, 이 상태에서는 동작을 멈추고, 위치와 속성등이 초기화 됩니다.

3. __GST_STATE_PAUSE__ : 이 상태에서 element는 스트림을 읽지만 적극적으로 프로세스가 진행되지 않습니다. element는 스트림 위치를 수정하고, 데이터를 읽고 처리하여 상태가 PLAYING으로 바뀌는 즉시 재생을 준비 할 수는 있지만, clock을 실행하게 하는 data를 재생할 수 없습니다. 다시말해, PAUSED 는 PLAYING과 같지만 clock이 작동되지는 않습니다.  
&nbsp; __PAUSED__ 상태로 진입한 element는 가능한 빨리 PLAYING 상태로 넘어가기 위해 준비를 해야한다. 예를들어 비디오와 오디오 출력은 도착한 데이터들을 대기열에 쌓아두어 상태변환 즉시 재생할 수 있도록 기다린다. 또한, 비디오 sink는 이미 첫 프레임에서 재생할 수 있다.(이는 clock에 영향을 받지 않기 때문이다.) Autoplugger들은 이 동일한 상태 전환을 사용하여 이미 파이프라인을 연결합니다. 그러나 코덱이나 filter 같은 다른 대부분의 element들은 이 상태에서 아무것도 수행 할 필요가 없습니다.

4. __GST_STATE_PLAYING__ : PLAYING 상태는 element는 clock이 작동한다는 것만 빼면 정확히 PAUSED 상태와 같습니다.


&nbsp; <span class="fill_color">gst_element_set_state()</span> 함수를 사용하여 element의 상태를 변경할 수 있습니다. 만약 element를 다른 상태로 변경한다면, Gstreamer는 내부적으로 모든 중간 상태를 통과합니다. 그래서 element를 NULL을 PLAYING으로 변경하면, Gstreamer은 내부적으로 element를 READY와 PAUSED 중간 상태로 설정합니다.

&nbsp; __GST_STATE_PLAYING__ 상태로 진입하면, 파이프라인은 데이터를 자동으로 처리합니다. 그들은 어떤 형태로든 반복될 필요가 없습니다. 내부적으로, Gstreamer는 이 작업을 수행하는 스레드를 시작할 것입니다. Gstreamer또한 GstBus를 이용하여 메세지를 파이프라인 스레드에서 application 자체 스레드로 전환하는 것을 처리할 것입니다. 자세한 내용은 [Bus](https://gstreamer.freedesktop.org/documentation/application-development/basics/bus.html?gi-language=c)를 참고하세요.

&nbsp; bin과 파이프라인을 특정 상태로 변경할 때, 프로그램은 이 상태변경을 bin과 파이프라인 내 모든 element에 자동적으로 전파하므로, 일반적으로 최상위 파이프라인의 상태를 설정하여 파이프라인을 시작하거나 중단하기만 하면 됩니다. 그러나 이미 실행중인 파이프라인에 element를 동적으로 추가하고자 할 때 ("pad-added" 신호 콜백 내에서), <span class="fill_color">gst_element_set_state()</span> 혹은 <span class="fill_color">gst_element_sync_state_with_parent()</span>함수를 사용하여 그 파이프라인을 직접 원하는 상태로 설정해야 합니다.


&nbsp; 이곳의 예제 코드들은 문서에서 자동적으로 추출되어 Gstreamer tarball 내부의 <span class="fill_color">tests/examples/manual</span>에서 빌드됩니다.
