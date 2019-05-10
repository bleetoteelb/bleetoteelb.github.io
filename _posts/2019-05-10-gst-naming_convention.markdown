---
layout: post
title:  "Gtype naming convention"
subtitle:   "Gtype naming convention"
categories: devlog
tags: gst
---

# __1. 개요__

GNOME Project에서 사용하는 Glib, Gobject등에서 사용하는 이름 짓기 규칙 인듯 하다.  
다음 링크에서 가져왔다.  
[gtype conventions](https://developer.gnome.org/gobject/stable/gtype-conventions.html)

<br>
<br>
# __2. Conventions__

- Object name을 포함한 Type name들은 =='a-z'==,'A-Z' 혹은 '_'로 시작하며 최소 3글자 이상이어야 한다.  

- Object_method 이름을 function 이름 짓는데에 사용하라. : 예를들어 method가 save이고 object tpye이 file이라면, file_save 이 된다.  

- 다른 프로젝트와의 함수 이름 충돌을 방지하기 위해, 모든 함수 앞에 library(or application)의 이름을 붙여라. : 예를들어 library가 viewer이고, save 동작을 한다면 함수명은 viewer_save 이 된다.

- 
<br>

