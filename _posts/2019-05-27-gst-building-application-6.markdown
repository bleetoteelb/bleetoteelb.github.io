---
layout: post
title:  "Building an Application(6)"
subtitle:   "Buffers and Events"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Buffers and Events__

&nbsp; 파이프라인을 흐르는 데이터들은 buffer와 event들로 가득차 있습니다. buffer는 실제 미디어 테이터를, event는 탐색 정보나 스트림종료 알림과 같은 제어 정보를 담고 있습니다. 이 모든 것들이 application이 실행 중일 때 파이프 라인을 통해 자동으로 흐릅니다. 이 파트는 주로 개념을 설명하기 위한 것이므로, 아무것도 하지 않고 읽어 내려가기만 하면 됩니다.

## __1. Buffers__

&nbsp; bufers는 만들어진 파이프라인을 통해 흘러가는 데이터를 담고 있습니다. source element는 보통 새로운 buffer를 만들고 체인 상 다음 element에 pad를 통해 전달합니다. 미디어 파이프라인을 생성하기 위해 Gstreamer 구조를 이용할 때, 직접 buffer를 처리할 필요는 없습니다. element가 알아서 처리해 줄 것입니다.

&nbsp; buffer는 다음 항목들을 포함하고 있습니다.

- 메모리 객체를 가리키는 포인터. 메모리 객체는 