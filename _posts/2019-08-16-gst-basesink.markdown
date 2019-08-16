---
layout: post
title:  "Concat plugin 분석"
subtitle:   "Concat plugin 뷴석"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# Introduction

 본 문서에서는 xvimagesink, filesink가 상속하는 element인 basesink 동작성을 분석해 보며 어떻게 이들이 구성되고 재생을 하는지 이해를 돕도록 한다.

## __1) Reference__

* * *
* [GstBaseSink Document](https://gstreamer.freedesktop.org/documentation/base/gstbasesink.html?gi-language=c)
* [gstbasesink.c](https://github.com/alessandrod/gstreamer/blob/master/libs/gst/base/gstbasesink.c)


## __2) 분석 POINT__

  
### __2-1) basesink 소개__

&nbsp; GstBaseSnk는 plugin element의 상위 단에서 여러 기능들을 제공하는 Gstreamer sink element의 기본 클래스이다. preroll, clock 동기화, state 변환, push/pull mode 활성화 등의 기능들을 제공한다.


preroll -> sync -> render