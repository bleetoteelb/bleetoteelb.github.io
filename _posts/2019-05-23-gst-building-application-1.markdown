---
layout: post
title:  "Building an Application(1)"
subtitle:   "Initializaing Gstreamer"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Initializing GStreamer__

&nbsp; Gstreamer application을 만들 때, 당신은 라이브러리 기능에 접근하기 위해 <span class="fill_color">gst/gst.h</span> 를 코드에 추가해주기만 하면 됩니다. 더불어, 당신은 GStreamer 라이브러리를 초기화 해주어야 합니다.

## __1. Simple initialization__

&nbsp; GStreamer 라이브러리들이 사용되기 전에, <span class="fill_color">gst_init</span>이 main에서 호출되어야 합니다. 이 호출로 라이브러리의 필수적인 초기화와 함께 GStreamer에 한정된 명령어 옵션(command line options)이 파싱 될 것입니다. 


다음은 초기화를 사용하는 일반적인 예제 코드입니다.
```c
#include <stdio.h>
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  const gchar *nano_str;
  guint major, minor, micro, nano;

  gst_init (&argc, &argv);

  gst_version (&major, &minor, &micro, &nano);

  if (nano == 1)
    nano_str = "(CVS)";
  else if (nano == 2)
    nano_str = "(Prerelease)";
  else
    nano_str = "";

  printf ("This program is linked against GStreamer %d.%d.%d %s\n",
          major, minor, micro, nano_str);

  return 0;
}
```
&nbsp; <span class="fill_color"> GST_VERSION_MAJOR</span>, <span class="fill_color"> GST_VERSION_MINOR</span> 그리고 <span class="fill_color"> GST_VERSION_MICRO</span> 매크로를 사용하여 작성중인 Gstreamer 버전을 가져오거나 <span class="fill_color"> gst_version </span> 함수를 사용하여 application에 연결된 Gstreamer 버전을 얻어올 수 있습니다. Gstreamer는 현재 같은 major와 minor 버전이라면 API 와 ABI가 호환이 되는 체계를 갖고 있습니다.

&nbsp; 물론 <span class="fill_color">gst_init</span>의 두 인자로 NULL값을 넘겨줄수도 있지만, 이 경우 명령어 옵션이 GStreamer에 의해 파싱 되지 않습니다.

<br>

## __2. The GOption interface__

&nbsp; GOption table을 이용하여 인자를 직접 초기화 할 수 있습니다. 다음은 이를 활용한 예제코드입니다.
```c
#include <gst/gst.h>

int
main (int   argc,
      char *argv[])
{
  gboolean silent = FALSE;
  gchar *savefile = NULL;
  GOptionContext *ctx;
  GError *err = NULL;
  GOptionEntry entries[] = {
    { "silent", 's', 0, G_OPTION_ARG_NONE, &silent,
      "do not output status information", NULL },
    { "output", 'o', 0, G_OPTION_ARG_STRING, &savefile,
      "save xml representation of pipeline to FILE and exit", "FILE" },
    { NULL }
  };

  ctx = g_option_context_new ("- Your application");
  g_option_context_add_main_entries (ctx, entries, NULL);
  g_option_context_add_group (ctx, gst_init_get_option_group ());
  if (!g_option_context_parse (ctx, &argc, &argv, &err)) {
    g_print ("Failed to initialize: %s\n", err->message);
    g_clear_error (&err);
    g_option_context_free (ctx);
    return 1;
  }
  g_option_context_free (ctx);

  printf ("Run me with --help to see the Application options appended.\n");

  return 0;
}
```

&nbsp; 위 예제코드에서 볼 수 있듯이, GOption table을 활용하여 application별로 다른 명령어 옵션을 정의하고, 이 table을 <span class="fill_color">gst_init_get_option_group</span>에서 반환된 옵션들과 함께 GLib 초기화 함수에 전달할 수 있습니다. Application 옵션들은 표준 Gstreamer 옵션들과 함께 파싱될것입니다.

&nbsp; 이곳의 예제 코드들은 문서에서 자동적으로 추출되어 Gstreamer tarball 내부의 <span class="fill_color">tests/examples/manual</span>에서 빌드됩니다.
