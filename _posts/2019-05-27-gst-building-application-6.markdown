---
layout: post
title:  "Building an Application - Buffers and Events"
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

- 메모리 객체를 가리키는 포인터. 메모리 객체는 데이터를 압축하여 메모리에 저장한다.
- buffer의 timestamp
- buffer를 사용하는 element의 개수를 나타내는 참조카운트. 이 참조카운트는 element가 더이상 사용되지 않을 때 해제하기 위해 사용됩니다.
- buffer flag

&nbsp; buffer는 생성되고, 메모리가 할당되고, 데이터가 저장됙, 다음 element에 전달됩니다. buffer를 넘겨받은 element는 데이터를 읽고 새로운 buffer를 만들어서 디코딩한 데이터를 저장하는 것과 같은 일들을 수행한뒤 기존의 buffer의 참조카운트를 하나 줄입니다. 참조 카운터가 줄어든 데이터는 (참조카운트가 0이라면) 해제되고 buffer는 제거됩니다. 일반적인 비디오와 오디오 디코더는 이렇게 동작합니다.

&nbsp; 이보다 더 복잡한 과정들도 있습니다. 예를들어 element는 새로운 buffer를 할당하지 않고 기존의 buffer를 수정할 수 있고, 하드웨어 메모리에 바로 데이터를 쓰거나(video-capture source처럼) X-server에서 할당받은 메모리에 데이터를 쓸 수도 있습니다(XShm을 이용해서). Buffer는 읽기전용으로 사용할 수 있습니다.


## __2. Events__

&nbsp; event는 buffer와 함께 파이프라인의 upstream과 downstream에서 보내지는 제어 부분입니다. downstream event는 같은 수준의 element에게 스트림의 상태를 알려줍니다. 탐색, 플러쉬, 스트림 종료 알림과 같은 event들이 있습니다. upstream event는 탐색처럼 스트림의 상태가 변화했을 때 요청하기 위해 application와 element 사이의 상호작용 뿐 아니라 element 사이에서의 상호작용에 사용됩니다. application에서는 오직 upstream evnet만이 중요하고, downstream event에서는 데이터 개념을 보다 완벽하게 파악하기 위해 설명됩니다.

&nbsp; 대부분의 application 탐색은 시간 단위로 이뤄지고, 아래 예제도 이와 같습니다.

```c
static void
seek_to_time (GstElement *element,
          guint64     time_ns)
{
  GstEvent *event;

  event = gst_event_new_seek (1.0, GST_FORMAT_TIME,
                  GST_SEEK_FLAG_NONE,
                  GST_SEEK_METHOD_SET, time_ns,
                  GST_SEEK_TYPE_NONE, G_GUINT64_CONSTANT (0));
  gst_element_send_event (element, event);
}
```

<span class="fill_color">gst_element_seek()</span>함수는 이를 위한 함수입니다.  위 예제코드는 단순히 어떻게 동작하는지 보여주기 위한 코드입니다.
(역주: 실제 <span class="fill_color">gst_element_seek()</span> 이라는 함수가 저렇게 정의되어 있으므로 저 동작을 원한다면 미리 정의되어 있는 <span class="fill_color">gst_element_seek()</span>함수를 쓰면 됩니다.)