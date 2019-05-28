---
layout: post
title:  "Advanced GStreamer concepts - Metadata"
subtitle:   "Metadata"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Metadata__

&nbsp; Gstreamer는 지원하는 두 가지 유형의 metadata를 명확히 구분하는데, 그 중 하나는 기술적이지 않은 방식으로 스트림의 내용을 서술하는 stream tag들이고 나머지 하나는 스트림의 속성의 기술적인 서술인 stream-info입니다. stream tag의 예시로는 노래의 저자, 노래 제목, 그 음원이 속해있는 앨범 등이 있고, stream-info의 예시로는 비디오 크기, 오디오 samplerate, 사용되는 코덱 등이 있습니다.

&nbsp; 태그들은 Gstreamer 태깅 시스템을 사용해 처리하고, stream-info는 그 pad의 현재 __GstCaps__ 로부터 얻은 __GstPad__ 에서 찾을 수 있습니다.

## __1. Metadata reading__

&nbsp; 스트림 정보는 __GstPad__ 로부터 정보를 불러 쉽게 얻을 수 있습니다. 이를 위해서는 스트림 정보가 필요한 모든 pad에 대한 접근이 필요합니다. 이 접근법에 대한 내용은 [Using capabilities for metadat](https://gstreamer.freedesktop.org/documentation/application-development/basics/pads.html?gi-language=c#using-capabilities-for-metadata)에 설명되어 있으니 지금은 넘어가도록 하겠습니다.

&nbsp; Tag 읽기는 Gstreamer의 bus를 통해 이뤄집니다. __GST_MESSAGE_TAG__ 메세지를 수신하고 원하는대로 처리할 수 있는데, 이것은 [Bus](https://gstreamer.freedesktop.org/documentation/application-development/basics/bus.html?gi-language=c)에서 이미 다룬 내용입니다.

&nbsp; 그러나 __GST_MESSAGE_TAG__ 메세지는 여러번 발생 할 수 있으며 일관된 방식으로 tag를 집계하고 표시하는것은 application의 책임입니다. 이 기능은 <span class="fill_color">gst_tag_list_merge()</span>를 사용해 할 수 있지만, 새로운 노래를 불러오거나 인터넷 라디오를 청취 할 때 몇 분마다 캐시를을 비워야 할 것입니다. 또한, 병합 모드로 <span class="fill_color">GST_TAG_MERGE_PREPEND</span>를 사용할 때에는 새 제목(나중에 받은 정보)이 이전 제목의 앞에 두어야 합니다.

&nbsp; 다음 예제코드는 파일에서 tag를 추출하고 그것을 출력하는 코드입니다.

```c
/* compile with:
 * gcc -o tags tags.c `pkg-config --cflags --libs gstreamer-1.0` */
#include <gst/gst.h>

static void
print_one_tag (const GstTagList * list, const gchar * tag, gpointer user_data)
{
  int i, num;

  num = gst_tag_list_get_tag_size (list, tag);
  for (i = 0; i < num; ++i) {
    const GValue *val;

    /* Note: when looking for specific tags, use the gst_tag_list_get_xyz() API,
     * we only use the GValue approach here because it is more generic */
    val = gst_tag_list_get_value_index (list, tag, i);
    if (G_VALUE_HOLDS_STRING (val)) {
      g_print ("\t%20s : %s\n", tag, g_value_get_string (val));
    } else if (G_VALUE_HOLDS_UINT (val)) {
      g_print ("\t%20s : %u\n", tag, g_value_get_uint (val));
    } else if (G_VALUE_HOLDS_DOUBLE (val)) {
      g_print ("\t%20s : %g\n", tag, g_value_get_double (val));
    } else if (G_VALUE_HOLDS_BOOLEAN (val)) {
      g_print ("\t%20s : %s\n", tag,
          (g_value_get_boolean (val)) ? "true" : "false");
    } else if (GST_VALUE_HOLDS_BUFFER (val)) {
      GstBuffer *buf = gst_value_get_buffer (val);
      guint buffer_size = gst_buffer_get_size (buf);

      g_print ("\t%20s : buffer of size %u\n", tag, buffer_size);
    } else if (GST_VALUE_HOLDS_DATE_TIME (val)) {
      GstDateTime *dt = g_value_get_boxed (val);
      gchar *dt_str = gst_date_time_to_iso8601_string (dt);

      g_print ("\t%20s : %s\n", tag, dt_str);
      g_free (dt_str);
    } else {
      g_print ("\t%20s : tag of type '%s'\n", tag, G_VALUE_TYPE_NAME (val));
    }
  }
}

static void
on_new_pad (GstElement * dec, GstPad * pad, GstElement * fakesink)
{
  GstPad *sinkpad;

  sinkpad = gst_element_get_static_pad (fakesink, "sink");
  if (!gst_pad_is_linked (sinkpad)) {
    if (gst_pad_link (pad, sinkpad) != GST_PAD_LINK_OK)
      g_error ("Failed to link pads!");
  }
  gst_object_unref (sinkpad);
}

int
main (int argc, char ** argv)
{
  GstElement *pipe, *dec, *sink;
  GstMessage *msg;
  gchar *uri;

  gst_init (&argc, &argv);

  if (argc < 2)
    g_error ("Usage: %s FILE or URI", argv[0]);

  if (gst_uri_is_valid (argv[1])) {
    uri = g_strdup (argv[1]);
  } else {
    uri = gst_filename_to_uri (argv[1], NULL);
  }

  pipe = gst_pipeline_new ("pipeline");

  dec = gst_element_factory_make ("uridecodebin", NULL);
  g_object_set (dec, "uri", uri, NULL);
  gst_bin_add (GST_BIN (pipe), dec);

  sink = gst_element_factory_make ("fakesink", NULL);
  gst_bin_add (GST_BIN (pipe), sink);

  g_signal_connect (dec, "pad-added", G_CALLBACK (on_new_pad), sink);

  gst_element_set_state (pipe, GST_STATE_PAUSED);

  while (TRUE) {
    GstTagList *tags = NULL;

    msg = gst_bus_timed_pop_filtered (GST_ELEMENT_BUS (pipe),
        GST_CLOCK_TIME_NONE,
        GST_MESSAGE_ASYNC_DONE | GST_MESSAGE_TAG | GST_MESSAGE_ERROR);

    if (GST_MESSAGE_TYPE (msg) != GST_MESSAGE_TAG) /* error or async_done */
      break;

    gst_message_parse_tag (msg, &tags);

    g_print ("Got tags from element %s:\n", GST_OBJECT_NAME (msg->src));
    gst_tag_list_foreach (tags, print_one_tag, NULL);
    g_print ("\n");
    gst_tag_list_unref (tags);

    gst_message_unref (msg);
  }

  if (GST_MESSAGE_TYPE (msg) == GST_MESSAGE_ERROR) {
    GError *err = NULL;

    gst_message_parse_error (msg, &err, NULL);
    g_printerr ("Got error: %s\n", err->message);
    g_error_free (err);
  }

  gst_message_unref (msg);
  gst_element_set_state (pipe, GST_STATE_NULL);
  gst_object_unref (pipe);
  g_free (uri);
  return 0;
}
```

## __2. Tag writing__

&nbsp; Tag 쓰기는 __GstTagSetter__ 인터페이스를 이용해 이뤄집니다. 이를 위해서는 파이프라인에서 tag-set-supporting element가 필요합니다.

&nbsp; 파이프라인 속 element가 tag 쓰기를 지원하는지 확인하기 위해서는 <span class="fill_color">gst_bin_iterate_all_by_interface(pipeline, GST_TYPE_TAG_SETTER)</span>함수를 사용하면 됩니다. 그 결과로 나온 element의 tag를 설정하기 위해서, 보통은 encoder 혹은 muxer에서 taglist는 <span class="fill_color">gst_tag_setter_merge_tags()</span>를, 개별적인 tag는 <span class="fill_color">gst_tag_setter_add_tags()</span>를 사용하면 됩니다. 

&nbsp; Gstreamer에서 지원하는 태그의 또 다른 장점은 태그들이 파이프라인 내에 보존된다는 것입니다. 즉, tag를 지원하는 미디어 타입으로 코드 변환하면 tag가 데이터 스트림의 일부로 다뤄지고 새롭게 쓰여지는 미디어 파일에 합쳐집니다.
