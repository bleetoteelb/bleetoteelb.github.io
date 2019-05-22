---
layout: post
title:  "About Gstreamer(2)"
subtitle:   "Design principles"
categories: devlog
tags: gst
---

<style>
.fill_color {background-color:rgba(164,164,164,0.7);border-radius:4px;padding:2px;}
.blue_l {color:#323C73;}
</style>

# __Design principles__

## __1. Clean and Powerful__  

&nbsp; Gstreamer는 다음의 대상들에게 깔끔한 인터페이스를 제공한다.

- __미디어 파이프라인을 만들고 싶은 application 개발자.__ 프로그래머는 단 한줄의 코드도 없이 강력한 Gstreamer 기능을 사용하여 미디어 파이프라인을 만들 수 있습니다. 또한 복잡한 미디어 조작을 손쉽게 할 수 있습니다.

- __플러그인 프로그래머.__ 플러그인 프로그래머들은 자체 포함된(?) 플러그인을 만들기 위해 깔끔하고 간단한 API를 제공받습니다. 거대한 디버깅과 추적 체계가 통합되어, Gstreamer는 예제로도 사용할 수 있는 실생활 플러그인들의 거대한 집합체가 되었습니다.
  
<br>

## __2. Object Oriented__  

&nbsp; Gstreamer는 _GObject_, _Glib 2.0 object model_ 에 기반합니다. GLib 2.0 혹은 GTK+와 친숙한 프로그래머들은 Gstreamer에 편하게 접근 할 수 있을 것입니다.  

&nbsp; Gstreamer는 시그널과 객체 속성 체계를 사용합니다. 

&nbsp; 모든 객체들은 다양한 속성과 기능을 위해 프로그램 실행중에 쿼리를 받을 수 있습니다.

&nbsp;  Gstreamer는 GTK+와 프로그래밍 방식이 비슷하도록 만들어졌습니다. 이 객체 모델, 객체 소유권, 참조 카운팅 등에 적용됩니다.


<br>

## __3. Extensible__  
 
&nbsp; 모든 Gstreamer 객체들은 GObject 상속 메소드를 사용하여 확장할 수 있습니다.

&nbsp; 모든 플러그인들은 동적으로 로드되고 독립적으로 확장 및 업그레이드 될 수 있습니다.

<br>

## __4. Allow binary-only plugins__
  
&nbsp; 플러그인들은 실행중에 로드되는 공유 라이브러리들입니다. 플러그인의 모든속성들이 GObject 속성들을 이용하여 설정될 수 있기 때문에, 플러그인들을 위한 헤더파일 설치가 필요가 없습니다. ( 사실상 설치할 방법이 없습니다. )

&nbsp; 완전히 자체 포함된 플러그인을 만들기 위해 모든 관련된 플러그인 요소들이 실행중에 쿼리를 받을 수 있습니다.

<br>

## __5. High performance__

&nbsp; Gstreamer의 고성능은 다음과 같은 방식으로 인해 이루어집니다.

- _GLib_ 의 _GSlice_ 할당자를 사용합니다.


- 플러그인 간에 매우 가볍게 연결됩니다. 데이터들은 최소한의 오버헤드를 위해 파이프라인을 지나갑니다. 플러그인들 사이에서의 데이터 전달은 일반적인 파이프라인에서 역참조 포인터만을 포함합니다.

- 타겟 메모리에서의 직접 작업하기 위한 체계를 제공합니다. 예를들어 플러그인은 X 서버의 공유된 메모리 공간에 직접 쓸 수 있습니다. 사운드 카드의 내부 하드웨어 버퍼들 처럼, 버퍼들은 임의의 메모리를 가리킬 수 있습니다.

- 참조 카운팅과 Copy-on-Write은 memcpy의 사용을 최소화합니다. 하위 버퍼들은 효율적으로 관리 가능한 조각들로 버퍼들을 분할합니다.

- 커널에 의해 관리되는 스케쥴링을 통해 전용 스트리밍 스레드들이 제공됩니다.

- 특수 프러그인들을 이용하여 하드웨어 가속을 허용합니다.

- 플러그인들의 스펙들과 플러그인 레지스트리를 이용하여 플러그인을 로딩하는 것이 실제 사용될 때까지 이루어지지 않습니다.

<br>

## __6. Clean core/plugins separation__

&nbsp; Gstreamer core는 미디어 지식과 관련되 부분이 없습니다. Gstreamer는 오직 btyes과 blocks에 대해서만 다루고 있고 기본적인 요소들만 포함합니다. Gstreamer core는 cp(유닉스 계열 운영체제의 복사 명령어)같은 low-level 시스템툴을 구현할 수 있을 만큼 충분히 기능적입니다.

&nbsp; 모든 미디어 처리 기능은 core 외부의 plugin들에 의해 제공됩니다. 이러한 사실들이 어떻게 Gstreamer core가 특수한 타입의 미디어들을 처리하는지를 잘 설명해줍니다.

<br>

## __7. Provide a framework for codec experimentation__

&nbsp; Gstreamer는 코덱 개발자들은 다른 알고리즘을 실험 할 수 있는 쉬운 프레임워크가 되길 원하며 _Xiph.Org Foundation(Theora 혹은 vorbis)_ 에서 개발한 것과 같은 무료 배포 멀티미디어 코덱의 개발을 지원합니다.
