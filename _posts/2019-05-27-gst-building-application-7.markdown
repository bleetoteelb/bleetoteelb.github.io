---
layout: post
title:  "Building an Application(7)"
subtitle:   "Your first application"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>


# __Your first application__

&nbsp; 이 파트에서는 이전 파트에서 배운 모든 내용을 요약할 것입니다. 초기화 라이브러리를 포함하고, element를 만들고, 파이프라인에서 element들을 묶고 pipeline을 재생하는 간단한 Gstreamer application의 모든 측면을 설명할 것입니다.이 과정이 끝나면, 당신은 간단한 Ogg/Vorbis audio player를 만들 수 있게됩니다.

## __Hello world__

&nbsp; 이제부터 간단한 Ogg/Vorbis command-line audio player를 첫 application으로 만들어볼것입니다. 이를 위해, 오로지 표준 Gstreamer만을 사용할 것이며, 플레이어는 command-line에 명시된 파일을 읽어들일 것입니다. 시작해 봅시다.

&nbsp; [Initializing Gstreamer](https://gstreamer.freedesktop.org/documentation/application-development/basics/init.html?gi-language=c)에서 우리는 application에서 제일 먼저 해야할 일이<span class="fill_color">gst_init()</span>을 호출하여 Gstreamer를 초기화하는 일이라고 배웠습니다. 그리고, application이 <span class="fill_color">gst/gst.h</span>를 포함하고 있는지 꼭 확인하십시오. 이를 포함하지 않으면 미리 정의된 gstreamer의 함수들을 사용할 수 없습니다. <span class="fill_color"># include <gst/gst.h></span>를 사용하십시오.

&nbsp; 다음으로, <span class="fill_color">gst_element_factory_make()</span>를 사용하여 다른 element를 생성하고 싶을 것입니다. Ogg/Vorbis audio player를 위해, 디스크로부터 파일을 읽어들이는 source element가 필요할 것입니다. Gstreamer는 "filesrc"라는 이름으로 element를 포함하고 있습니다. 다음으로, 파일을 분석하고 audio 원본데이터를 디코딩해야 합니다. Gstreamer는 이를 위한 두가지 element가 있는데 첫번째는 "oggdemux"라고 불리는 Ogg 스트림을 기본스트림(비디오, 오디오)로 분석해주는 element이며, 두번째는 "vorbisdec"라는 편리한 Vorbis 오디오 디코더입니다. "oggdemux"는 각각의 기본스트림에 동적 pad를 생성하기 때문에, Ogg demuxer와 Vorbis 디코더 element를 링크해주기 위해 [Dynamic(or sometimes)](https://gstreamer.freedesktop.org/documentation/application-development/basics/pads.html?gi-language=c#dynamic-or-sometimes-pads)에서 배웠던 것처럼 "oggdemux" element에 "pad-added" 이벤트 처리기를 설정해주어야 합니다. 마지막으로, 오디오 output element가 필요하기 때문에, 오디오 장치를 자동으로 탐색해주는 "autoaudiosink"를 사용할 것입니다.

&nbsp; 마지막으로 남은것은 container element인 __GstPipeline__ 에 모든 element를 추가하고 전체 노래가 재생될 때까지 기다리는 것입니다. 이전에 [Bins](https://gstreamer.freedesktop.org/documentation/application-development/basics/bins.html?gi-language=c)에서 container bin에 element를 어떻게 추가하는지를 배웠고, [Element States](https://gstreamer.freedesktop.org/documentation/application-development/basics/elements.html?gi-language=c#element-states)에서는 element state를 배웠습니다. 우리는 error를 받거나 스트림의 종료를 감지하기 위해 pipeline에 message handler 역시 추가할 것입니다.

&nbsp; audio player의 코드 전문을 아래에 첨부합니다.

```c
#include <gst/gst.h>
#include <glib.h>


static gboolean
bus_call (GstBus     *bus,
          GstMessage *msg,
          gpointer    data)
{
  GMainLoop *loop = (GMainLoop *) data;

  switch (GST_MESSAGE_TYPE (msg)) {

    case GST_MESSAGE_EOS:
      g_print ("End of stream\n");
      g_main_loop_quit (loop);
      break;

    case GST_MESSAGE_ERROR: {
      gchar  *debug;
      GError *error;

      gst_message_parse_error (msg, &error, &debug);
      g_free (debug);

      g_printerr ("Error: %s\n", error->message);
      g_error_free (error);

      g_main_loop_quit (loop);
      break;
    }
    default:
      break;
  }

  return TRUE;
}


static void
on_pad_added (GstElement *element,
              GstPad     *pad,
              gpointer    data)
{
  GstPad *sinkpad;
  GstElement *decoder = (GstElement *) data;

  /* We can now link this pad with the vorbis-decoder sink pad */
  g_print ("Dynamic pad created, linking demuxer/decoder\n");

  sinkpad = gst_element_get_static_pad (decoder, "sink");

  gst_pad_link (pad, sinkpad);

  gst_object_unref (sinkpad);
}



int
main (int   argc,
      char *argv[])
{
  GMainLoop *loop;

  GstElement *pipeline, *source, *demuxer, *decoder, *conv, *sink;
  GstBus *bus;
  guint bus_watch_id;

  /* Initialisation */
  gst_init (&argc, &argv);

  loop = g_main_loop_new (NULL, FALSE);


  /* Check input arguments */
  if (argc != 2) {
    g_printerr ("Usage: %s <Ogg/Vorbis filename>\n", argv[0]);
    return -1;
  }


  /* Create gstreamer elements */
  pipeline = gst_pipeline_new ("audio-player");
  source   = gst_element_factory_make ("filesrc",       "file-source");
  demuxer  = gst_element_factory_make ("oggdemux",      "ogg-demuxer");
  decoder  = gst_element_factory_make ("vorbisdec",     "vorbis-decoder");
  conv     = gst_element_factory_make ("audioconvert",  "converter");
  sink     = gst_element_factory_make ("autoaudiosink", "audio-output");

  if (!pipeline || !source || !demuxer || !decoder || !conv || !sink) {
    g_printerr ("One element could not be created. Exiting.\n");
    return -1;
  }

  /* Set up the pipeline */

  /* we set the input filename to the source element */
  g_object_set (G_OBJECT (source), "location", argv[1], NULL);

  /* we add a message handler */
  bus = gst_pipeline_get_bus (GST_PIPELINE (pipeline));
  bus_watch_id = gst_bus_add_watch (bus, bus_call, loop);
  gst_object_unref (bus);

  /* we add all elements into the pipeline */
  /* file-source | ogg-demuxer | vorbis-decoder | converter | alsa-output */
  gst_bin_add_many (GST_BIN (pipeline),
                    source, demuxer, decoder, conv, sink, NULL);

  /* we link the elements together */
  /* file-source -> ogg-demuxer ~> vorbis-decoder -> converter -> alsa-output */
  gst_element_link (source, demuxer);
  gst_element_link_many (decoder, conv, sink, NULL);
  g_signal_connect (demuxer, "pad-added", G_CALLBACK (on_pad_added), decoder);

  /* note that the demuxer will be linked to the decoder dynamically.
     The reason is that Ogg may contain various streams (for example
     audio and video). The source pad(s) will be created at run time,
     by the demuxer when it detects the amount and nature of streams.
     Therefore we connect a callback function which will be executed
     when the "pad-added" is emitted.*/


  /* Set the pipeline to "playing" state*/
  g_print ("Now playing: %s\n", argv[1]);
  gst_element_set_state (pipeline, GST_STATE_PLAYING);


  /* Iterate */
  g_print ("Running...\n");
  g_main_loop_run (loop);


  /* Out of the main loop, clean up nicely */
  g_print ("Returned, stopping playback\n");
  gst_element_set_state (pipeline, GST_STATE_NULL);

  g_print ("Deleting pipeline\n");
  gst_object_unref (GST_OBJECT (pipeline));
  g_source_remove (bus_watch_id);
  g_main_loop_unref (loop);

  return 0;
}
```

&nbsp; 완전한 pipeline을 생성했으니, 도식화된 그림으로 확인해봅시다.  
![hello-world](https://bleetoteelb.github.io/assets/img/hello-world.png)


<br>

## __2. Compiling and Running helloworld.c__

&nbsp; helloworld example을 컴파일 하기 위해 다음 명령어를 사용하세요.  
```bash
gcc -Wall helloworld.c -o helloworld $(pkg-config --cflags --libs gstreamer-1.0)
```

&nbsp; Gstreamer에서는 application 컴파일에 필요한 compiler와 linker flag를 얻기 위해 pkg-config를 사용합니다. 

&nbsp; 비표준 설치를 실행한다면(기존에 구성된 패키지를 사용하는 대신에 source로부터 Gstreamer를 직접 설치하는 경우), __PKG_CONFIG_PATH__ 환경변수가 정확하게 설정되어 있는지를 확인하십시오. (__$libdir/pkgconfig__)

&nbsp; 이와 다르게 비설치 Gstreamer setup을 사용한다면(gst-uninstalled같은), helloworld를 빌드하기 위해 __libtool__ 을 사용할 수도 있습니다. 
```bash
libtool --mode=link gcc -Wall helloworld.c -o helloworld $(pkg-config --cflags --libs gstreamer-1.0)
```

&nbsp; <sapn class="fill_color">./helloworld file.ogg</sapn>를 이용해 예제를 돌릴경우, __file.ogg__ 은 당신이 가장 좋아하는 Ogg/Vorbis file로 사용하십시오.

<br>

## __3. Conclusion__

&nbsp; 위에서 설명했듯이, 파이프라인을 설정하는것은 low-level이지만 강력합니다. 이후에 [Higher-level interfaces for Gstreamer applications](https://gstreamer.freedesktop.org/documentation/application-development/highlevel/index.html?gi-language=c)에서 더 높은 level의 인터페이스를 이용하여 더 적은 노력으로도 더 강력한 미디어 플레이어를 생성하는 메뉴얼에 대해 보게 될 것입니다. 그러나 그에 앞서서 응용 내부 Gstreamer에 대해 더 알아봅시다.

&nbsp; 우리가 위에서 살펴본 예제코드는 데스크탑 환경에서 네트워크에서 데이터를 읽어들이는 다른 element나 더 잘 통합된 data source element로 손쉽게 대체될 수 있다는 것을 잊지마십시오. 또한, 다른 미디어타입을 지원하는 decoder와 parser/demuxer를 사용할 수도 있습니다. Linux가 아닌 다른 운영체제(mac OS X, window, FreeBSD)에서는 다른 audio sink를 사용할 수 있습니다. audio card source를 사용하여, 재생대신 audio capture를 할 수도 있습니다. 이 모든 것들이 Gstreamer의 장점인 Gstreamer element의 재사용성을 보여줍니다.