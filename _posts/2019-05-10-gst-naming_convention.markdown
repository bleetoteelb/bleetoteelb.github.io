---
layout: post
title:  "Gtype naming convention"
subtitle:   "Gtype naming convention"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __1. 개요__

GNOME Project에서 사용하는 Glib, Gobject등에서 사용하는 이름 짓기 규칙 인듯 하다.  
다음 링크에서 가져왔다.  
[gtype conventions](https://developer.gnome.org/gobject/stable/gtype-conventions.html)

<br>

<br>

# __2. Conventions__

- Object name을 포함한 Type name들은 <span class="fill_color">'a-z'</span>,<span class="fill_color">'A-Z'</span> 혹은 <span class="fill_color">'_'</span>로 시작하며 최소 3글자 이상이어야 한다.  

- <span class="fill_color">Object_method</span> 이름을 function 이름 짓는데에 사용하라. : 예를들어 method가 <span class="fill_color">save</span>이고 object tpye이 <span class="fill_color">file</span>이라면, <span class="fill_color">file_save</span> 이 된다.  

- 다른 프로젝트와의 함수 이름 충돌을 방지하기 위해, 모든 함수 앞에 library(or application)의 이름을 붙여라. : 예를들어 library가 <span class="fill_color">viewer</span>이고, save 동작을 한다면 함수명은 <span class="fill_color">viewer_save</span> 이 된다.

- Gtype을 반환하는 매크로를 정의 할 때 <span class="fill_color">PREFIX_TYPE_OBJECT</span>으로 사용하라. 만약 object가 file이고 Viewer namespace에 포함되어있다면, <span class="fill_color">VIEWER_TYPE_FILE</span> 으로 쓰면 된다. 함수 이름에도 <span class="fill_color">prefix_object_get_type</span> 처럼 쓸 수 있다. (예 : <span class="fill_color">viewer_file_get_type</span>)

- 객체의 다양한 다른 관습 매크로를 정의하기 위해 <span class="fill_color blue_l">G_DELCARE_FINAL_TYPE</span> 혹은 <span class="fill_color blue_l">G_DECLARE_DERIVABLE_TYPE</span>을 사용하라.

    - <span class="fill_color">PREFIX_OBJECT (obj)</span> : PrefixObject 타입의 포인터를 반환한다.
    - <span class="fill_color">PREFIX_OBJECT_CLASS (klass)</span> : PrefixObjectClass 타입의 클래스 구조에 대한 포인터를 반환한다. (예 : <span class="fill_color">VIEWER_FILE_CLASS</span>)
    - <span class="fill_color">PREFIX_IS_OBJECT (obj)</span> : 입력 객체 인스턴스 포인터가 NULL이 아니고 type OBJECT인지를 boolean으로 반환한다.
    - <span class="fill_color">PREFIX_IS_OBJECT_CLASS (klass)</span> : 입력 클래스 포인터가 type OBJECT인지를 boolean으로 반환한다.
    - <span class="fill_color">PREFIX_OBJECT_GET_CLASS (obj)</span> : 주어진 type의 인스턴스의 클래스 포인터를 반환한다. 

- 파일명을 지을 때 작명 방식에는 3가지가 있다.
    - hypen을 이용한 방식 : <span class="fill_color">viewer-file.h</span> <span class="fill_color">viwer-file.c</span> (Nautilus와 대부분의 GNOME library에서 사용)
    - underscore를 이용한 방식 : <span class="fill_color">viewer_file.h</span> <span class="fill_color">viewer_file.c</span>
    - 두 단어를 구분하지 않는 방식 : <span class="fill_color">fiwerfile.h</span> <span class="fill_color">viwerfile.c</span> (GTK+에서 convention)

<br>

<br>

# 3. __사용예시__

```c
#define VIEWER_TYPE_FILE viewer_file_get_type ()
G_DECLARE_FINAL_TYPE (ViewerFile, viewer_file, VIEWER, FILE, GObject)
```
별다른 요구가 없다면 <span class="fill_color">G_DEFINE_TYPE</span> 매크로를 이용해서 다음과 같이 줄일 수 있다.

```c
G_DEFINE_TYPE (ViewerFile, viewer_file, G_TYPE_OBJECT)
```
그렇지 않다면 다음과 같이 <span class="fill_color">viewer_file_get_type</span>을 직접 구현해줘야 한다.

```c
GType viewer_file_get_type (void)
{
  static GType type = 0;
  if (type == 0) {
    const GTypeInfo info = {
      /* You fill this structure. */
    };
    type = g_type_register_static (G_TYPE_OBJECT,
                                   "ViewerFile",
                                   &info, 0);
  }
  return type;
}
```