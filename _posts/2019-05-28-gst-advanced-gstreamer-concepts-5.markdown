---
layout: post
title:  "Advanced GStreamer concepts(5)"
subtitle:   "Buffering"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Buffering__

&nbsp; 버퍼링의 목적은 파이프라인에 충분한 데이터를 축적하여 재생이 끊김없이 부드럽게 되도록 함에 있습니다. 이는 보통 느리고 라이브가 아닌 네트워크 source에서 데이터를 받을 때 사용되지만 라이브 source에서도 사용될 수 있습니다.

&nbsp; Gstreamer는 다음의 기능들을 제공합니다.

- 재생을 시작하기 전에 메모리에 일정한 양의 데이터만큼 버퍼를 채워서 네트워크 변동을 최소화합니다. 아래의 __Stream buffering__ 부분을 참조하십시오.

- 네트워크 파일을 로컬 디스크에 저장하여 다운로드된 데이터에서 빠르게 탐색하도록 합니다. 이는 quicktime/youtube player와 비슷합니다. 아래의 __Download buffering__ 부분을 참조하십시오.

- 