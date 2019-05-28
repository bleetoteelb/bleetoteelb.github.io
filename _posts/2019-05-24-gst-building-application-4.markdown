---
layout: post
title:  "Building an Application - Bus"
subtitle:   "Bus"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Bus__

&nbsp; bus는 그것이 포함되어있는 자체 스레드 컨텍스트 안에서 스트리밍 스레드에서 application로 메세지를 전달하는것을 관리하는 간단한 시스템입니다. bus의 장점은 Gstreamer가 매우 많은 스레드를 형성한 경우에도, Gstreamer를 사용하기 위해 application 스레드를 인식하지 않아도 된다는 점입니다.

&nbsp; 모든 파이프라인은 bus를 기본적으로 갖고 있어서 application은 bus를 따로 생성하지 않아도 됩니다. application는 signal handler를 object에 연결하는 것처럼, bus에 message handler를 연결해 주기만 하면 됩니다. mainloop가 실행될 때, bus는 주기적으로 새로운 메세지를 체크해서 message가 있으면 콜백함수를 실행합니다.

## __1. How to use a bus__

&nbsp; bus를 사용하는 방법에는 두가지가 있습니다.

1. GLib/Gtk+ mainloop를 실행하고(혹은 주기적으로 기본 GLib main context를 반복할수도 있습니다.) bus에 몇가지 watch를 추가합니다. 이 방법에서 GLib main loop는 bus의 새 메세지를 확인하고 message가 있을 때마다 알려줍니다.  
&nbsp; 보통 이 경우에 <span class="fill_color">gst_bus_add_watch()</span>혹은 <span class="fill_color">gst_bus_add_signal_watch()</span>를 사용합니다.  
&nbsp; bus를 사용하기 위해서  <span class="fill_color">gst_bus_add_watch()</span>를 사용하여 message handler를 파이프라인의 bus에 붙입니다. 이 handler에서 signal 타입을 확인하고 그에 따라 어떤 수행이 이루어집니다. handler의 반환값으로 TRUE이면 handler를 bus에 계속 붙여두고, FALSE면 제거합니다.

2. bus가 자체적으로 message를 확인합니다. 이는 <span class="fill_color">gst_bus_peek()</span> 혹은 <span class="fill_color">gst_bus_poll()</span>을 사용해 수행할 수 있습니다.

```c
#include <gst/gst.h>

static GMainLoop *loop;

static gboolean
my_bus_callback (GstBus * bus, GstMessage * message, gpointer data)
{
  g_print ("Got %s message\n", GST_MESSAGE_TYPE_NAME (message));

  switch (GST_MESSAGE_TYPE (message)) {
    case GST_MESSAGE_ERROR:{
      GError *err;
      gchar *debug;

      gst_message_parse_error (message, &err, &debug);
      g_print ("Error: %s\n", err->message);
      g_error_free (err);
      g_free (debug);

      g_main_loop_quit (loop);
      break;
    }
    case GST_MESSAGE_EOS:
      /* end-of-stream */
      g_main_loop_quit (loop);
      break;
    default:
      /* unhandled message */
      break;
  }

  /* we want to be notified again the next time there is a message
   * on the bus, so returning TRUE (FALSE means we want to stop watching
   * for messages on the bus and our callback should not be called again)
   */
  return TRUE;
}

gint
main (gint argc, gchar * argv[])
{
  GstElement *pipeline;
  GstBus *bus;
  guint bus_watch_id;

  /* init */
  gst_init (&argc, &argv);

  /* create pipeline, add handler */
  pipeline = gst_pipeline_new ("my_pipeline");

  /* adds a watch for new message on our pipeline's message bus to
   * the default GLib main context, which is the main context that our
   * GLib main loop is attached to below
   */
  bus = gst_pipeline_get_bus (GST_PIPELINE (pipeline));
  bus_watch_id = gst_bus_add_watch (bus, my_bus_callback, NULL);
  gst_object_unref (bus);

  /* [...] */

  /* create a mainloop that runs/iterates the default GLib main context
   * (context NULL), in other words: makes the context check if anything
   * it watches for has happened. When a message has been posted on the
   * bus, the default main context will automatically call our
   * my_bus_callback() function to notify us of that message.
   * The main loop will be run until someone calls g_main_loop_quit()
   */
  loop = g_main_loop_new (NULL, FALSE);
  g_main_loop_run (loop);

  /* clean up */
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  g_source_remove (bus_watch_id);
  g_main_loop_unref (loop);

  return 0;
}
```

&nbsp; mainloop의 스레드 context에서 handler가 호출된다는 것을 아는 것은 중요합니다. 이는 bus에서 파이프라인과 application의 상호작용이 비동기이며 실시간 목적에 맞지 않아, 이론적으로 끊김없는 재생과 비디오에 영향을 미치기 때문입니다. 모든 그러한 것들은 Gstreamer 프러그인을 제작하는 것이 가장 쉬운 파이프라인 context에서 이루어져야 합니다. 이는파이프라인에서 application으로 메세지를 전달하는 주된 목적에 매우 유용합니다. 이러한 접근법의 장점은 Gstreamer 내부적으로 이뤄지는 스레드 생성이 application에서 숨겨지고 application 개발자가 스레드 문제를 전혀 염려 할 필요가 없다는 점입니다.

&nbsp; 기본 GLib mainloop을 사용한다면 watch를 붙이는 대신에 "message" 시그널을 bus에 연결할 수 있습니다. 이 방법을 사용하면 __switch()__ 으로 모든 가능한 메세지 타입을 바꿀 필요가 없습니다. 단지 "message::\<type>"의 형식으로 관심 있는 시그널을 연결하면 되고, 여기서 \<type>은 특별한 메세지 타입입니다.

&nbsp; 위 내용은 다음과 같이 쓰여질 수 있습니다.
```c
GstBus *bus;

[..]

bus = gst_pipeline_get_bus (GST_PIPELINE (pipeline));
gst_bus_add_signal_watch (bus);
g_signal_connect (bus, "message::error", G_CALLBACK (cb_message_error), NULL);
g_signal_connect (bus, "message::eos", G_CALLBACK (cb_message_eos), NULL);

[..]
```

&nbsp; GLib mainloop를 사용하지 않는다면, 비동기 메세지 시그널은 기본으로 사용할 수 없습니다. 그러나 custom sync handler를 설치하고 시그널을 보내기 발생시키기 위해 <span class="fill_color">gst_bus_async_siganl_func() </span>을 사용할 수 있습니다. (자세한 내용은 [documentation](https://gstreamer.freedesktop.org/data/doc/gstreamer/stable/gstreamer/html/GstBus.html)을 참고하세요)

## __2. Message types__

&nbsp; Gstreamer에는 bus를 통해 전달할 수 있는 이미 정의된 message 타입을 갖고있습니다. 그러나 이 메세지들은 확장 가능하여 플러그인들은 추가적인 메세지들을 정의할 수 있고, application은 메세지를 위한 특정코드를 가지거나 무시할 수 있습니다. 모든 application은 사용자들에게 시각적인 피드백을 제공하기 위해 최소한 에러 메세지정도는 처리하는 것이 좋습니다.

&nbsp; 모든 메세지들이 message source, type 그리고 timestamp를 갖고 있습니다. message source를 사용하여 어떤 element가 메세지를 보내는지 확인할 수 있습니다. 예를들어 일부 메세지의 경우 최상위 수준 파이프라인에서 보내는 메세지들만이 application에서 필요할 수도 있습니다. (예를들어, 상태변화 알림과 같은 시그널) 다음은 모든 메세지의 목록과 메세지별 내용을 어떻게 처리하는지와 그 메세지들이 무엇을 하는지에 대한 간단한 설명들입니다.

- __Error, warning and information notification__ : 메세지가 파이프라인의 상태에 대해 사용자들에게 표시되어야 하는 경우, element에 의해 이 메세지들이 사용됩니다. 에러메세지들은 치명적이고 데이터 분석을 종료합니다. 파이프라인 활동을 재시작하기 위해서는 반드시 이 오류를 복구해야 합니다. 경고 메세지들은 치명적이진 않지만, 그럼에도 불구하고 문제를 암시합니다. 정보 메세지들은 문제가 되지 않는 알림들입니다. 이 모든 메세지들은 main 오류 타입 및 메세지가 있는 GError와 선택적으로 디버그 문자열을 포함합니다. 두 종류 모두 <span class="fill_color">gst_message_parse_error()</span>, <span class="fill_color">_parse_warning()</span> 그리고 <span class="fill_color">_parse_info()</span>를 이용해 추출할 수 있습니다. 에러내용과 디버그 문자열은 사용후에 반드시 해제해야 합니다.

- __End-of-stream notification__ : 스트림이 종료될 때 이 메세지를 발생합니다. 파이프라인의 상태는 바뀌지 않지만, 추가 미디어 스트림의 처리가 중단됩니다. application은 이 메세지를 재생목록에서 다음 노래로 넘어갈 때 사용합니다. 스트림 종료 이후, 스트림 내 한 지점을 찾아 돌아갈 수도 있습니다. 재생은 자동적으로 계속됩니다. 이 메세지는 특별한 인자를 갖고있지는 않습니다.

- __Tags__ : 이 메세지는 스트림에서 metadata가 발견되었을 때 발생합니다. 이 메세지는 파이프 라인에 여러번 표시될 수 있습니다. (예: 아티스트 이름이나 노래 제목과 같은 설명 metadata 및 샘플 정보 및 비트 전송률과 같은 스트림 정보의 경우 한번) application은 metadata를 내부적으로 임시저장합니다. <span class="fill_color">gst_message_parse_tag()</span>는 taglist를 사용하는데에 사용되어야 하고, 더 이상 사용되지 않는 임시저장된 데이터는 <span class="fill_color">gst_tag_list_unref</span>를 이용하여 참조를 해제해야 합니다.

- __State-changes__ : 이 메세지는 상태변경에 성공했을 때 발생합니다. <span class="fill_color">gst_message_parse_state_changed()</span>를 사용하여 이전상태와 새로운 상태를 분석할 수 있습니다.

- __Buffering__ : 이 메세지는 네트워크 스트림을 임시저장하는 동안 발생합니다. <span class="fill_color">gst_message_get_structure()</span>함수가 반환한 구조체에서 "buffer-percent" 속성을 추출한 메세지로부터 백분율 단위의 진행상태를 수동으로 표시할 수 있습니다. 버퍼링에 관련한 자세한 내용은 [Buffering](https://gstreamer.freedesktop.org/documentation/application-development/advanced/buffering.html?gi-language=c) 문서를 참조하십시오.

- __element messages__ : 이 메세지들은 특정 element에 특별하고 보통은 추가 기능을 나타내는 특수한 메세지입니다. element의 설명 문서에서는 특정 element가 보낼 수 있는 element 메세지를 상세히 써 두어야 합니다. 예를들어, 'qtdemux' QuickTime demuxer element는 스트림에서 redirect 명령을 포함하고 있을 때 'redirect' element 메세지를 보낼 수 있습니다.

- __Application-specific messages__ : 메세지 구조를 얻고 그 구조를 읽음으로써 추출할 수 있는 해당 정보들을 담고 있는 메세지 입니다. 일반적으로 이 메세지는 무시해도 됩니다.

application 메세지들은 주로 application이 어떤 스레드로부터 메인스레드로 정보를 모아야 하는 경우에 내부에서 사용하기 위해 만들어졌습니다. 이는 스트리밍 스레드의 컨텍스트에서 이러한 시그널들이 발생하기 때문에 application이 element signal을 사용할 때 특히 유용합니다. 