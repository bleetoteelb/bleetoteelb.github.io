---
layout: post
title:  "정리하기 북마크용 페이지"
subtitle:   "정리하기 북마크용 페이지"
categories: devlog
tags: gst
---


이 페이지는 공부하다가 정리가 잘 되어 있는 링크나  
나중에 나도 따로 정리 할 링크가 있을 경우 모아놓는 페이지다.

언제 무엇을 정리할지 모르므로, 넘버링은 하지 않고

계속해서 덧붙히다가  
너무 많아진다 싶으면 새로 포스트 만들고 그럴목적 ㅇㅇ


# make 파일

make 파일 문법을 잘 정리되어 있는 것 같아서 북마크.
나중에 제대로 정리해서 make 관련 되어서도 정리해볼까 생각중

[make파일 링크](https://wiki.kldp.org/KoreanDoc/html/gcc_and_make/gcc_and_make-3.html)  
[또다른 make파일 링크](https://bowbowbow.tistory.com/12?category=176521)

<details>
<summary>리눅스 강의 내용</summary>
<div markdown="1">
tail –n +숫자 => 숫자에 해당하는 행부터 끝까지 출력  
tail –n –숫자 -> 숫자에 절대값만큼의 행만 출력  
전체 프로세스 상태 및 자원 활용량 확인 (ps aux, top)  
프로세스의 부모/자식 관계 확인 (pstree, ps axjf)  
프로세스의 상태 확인 (ps aux, cat /proc/[PID]/status, ps axeo pid, stat, comm)  

디렉터리/파일 복사 (cp [option] [원본] [대상파일] or [디렉터리])  
-a 가능한 원래 파일의 구조와 속성을 그대로 복사  
-b 복사할 때 덮어쓰게 되는 파일은 백업 생성  
-d 심볼릭 링크는 심볼릭 링크로 복사  
-f 복사 위치에 존재하는 파일을 제거하고 복사  
-i 복사 시 같은 이름의 파일이 존재하면 덮어 쓸 것인가 확인  
-l 하드링크를 만듦  
-P 원본파일의 소유자,권한,시간 기록을 그대로 복사  
-r,-R 파일과 서브 디렉터리에 포함된 모든 파일을 recursive하게 복사  
-s 디렉터리가 아닌 파일의 심볼릭 링크 만들기  
-u 파일 정보 갱신  
-x 다른 파일 시스템인 하위 디렉터리는 무시  
 
디렉터리/파일 이동/이름 바꿈 (mv [option] [원본] [대상파일] or [디렉터리])  
-b 이동할 파일이 이미 존재한다면 백업파일을 만듦  
-f 복사 위치에 존재하는 파일을 덮어씀  
-i 이동 시 같은 이름의 파일이 존재하면 덮어 쓸 것인가 확인  
-n 이동 시 같은 이름의 파일은 덮어 쓰지 않음  
-u 파일 정보 갱신  

디렉토리/파일 삭제 -rm [option] [파일명]  
-f 삭제 할 때 삭제확인을 하지 않음  
-r 파일과 서브 디렉터리에 포함된 모든 파일을 recursive하게 삭제  
-i 모든 파일을 삭제하기 전에 확인  
-I 3개 이상의 파일을 삭제할 때 확인  
-v 진행상태를 표시함  

파일 관련 명령어 –chmod  
-c 변경된 결과를 표시함  
-f 에러 메시지를 표시하지 않음  
-v 모든 파일에 대한 처리 상황을 표시  
-R 하위 디렉터리와 파일에 적용  

-rwxrwxrwx 왼쪽부터 파일 소유자의 권한 / 파일 소유자가 속해 있는 그룹의 권한 / 그 이외 모든 사람들의 권한  
u:owner, g:group, o:other, a:all user  
+권한추가,-권한삭제  
https://conory.com/blog/19194 < 나중에 다시 읽고 정리  

현재 디렉터리 확인 –pwd  
디렉터리 내용 출력 –ls [option]  
-a ‘.’로 시작되는 파일을 포함해 모든 파일을 보여줌  
-d 현재 디렉터리에 대한 정보 출력   
-l 각 파일들의 소유자,권한, 갱신일 등 자세한 정보를 보여줌  
-s 파일이 얼마나 많은 디스크 블록을 차지하고 있는가를 보여줌  
-t 파일 갱신일 순서로 정렬  
-u Access한 날짜 순서로 정렬  
-c i-node가 마지막 바뀐 시간 순서로 정렬  
-r 정렬된 순서의 역으로 출력  
-I 파일의 i-node번호를 출력  
-C 열의 엔트리 출력. 정렬방식을 세로로함  
-F 파일의 특성 문자를 출력  
-R Recursive하게 출력->현 디렉터리 뿐 아니라 모든 서브 디렉터리까지 출력  

gcc hello.c  < hello.c 컴파일  
gcc –o hello hello.c < -o는 hello.c의 컴파일 결과물의 이름을 지정해 줄 때 씀  
gcc –g hello.c < -g는 실행파일에 디버깅 정보를 저장할 때 씀  
gcc –l 링크할 라이브러리를 지정  
gcc –I include 할 헤더파일을 찾을 디렉터리를 지정  
gcc –L 링크할 라이브러리 파일을 찾을 디렉터리를 지정  
gcc –c 여러 개의 소스파일을 분할컴파일 하기 위해 오브젝트 파일만 만들 때 사용  

cross compiler  
hello.c  
arm-gnueabi-linux-gcc –o hello hello.c < 컴파일러 –x86linux 환경에서 실행됨  
hello < 컴파일 된 실행파일 – arm linux 환경에서 실행됨  

Make란?  
프로그램 빌드 자동화 소프트웨어  
유닉스에서 가장 중요한 도구 중 하나  
gmake, gnumake가 더 향상된 도구임  
프로그램의 기능과 구조가 복잡해져 실행파일을 만들기 이ㅜ한 절차와 방법이 복잡해질 경우 Makefile을 이용하여 실행 파일을 만들 수 있도록 하는 명령  
Makefile / makeifle / GNUMakefile 세 개 파일 중 하나라도 있어야 한다.  
실행하면 default로 Makefile 또는 makefile이라는 이름의 파일을 찾고, 다른 이름을 준 경우는 –f 옵션으로 파일 이름을 지정한다.  
여러 파일들 간의 의존성과 각 파일을 위한 명령어를 정의한 Makefile을 해석하여 프로젝트를 빌드한다.  

Target: Dependencies  
	Commands  
Target – Command를 실행한 결과로 만들어 질 목적 파일 (object 및 실행 파일이 오며, all, clean과 같은 레이블이 올 수 있음)  




</div>
</details>
.

.


# 가변인자


가변인자(가변 파라미터) 라는건 처음 봤다.
설명이 잘 되어있는 것 같아서 북마크

문법에 관련돼서도 나중에 조금씩 정리를 해볼예정

[가변인자 링크](https://norux.me/19)


# Legacy 방송 / DVR 서비스

Legacy란 과거로부터 물려 내려온것.  
[Legacy 미디어 디지털 전략의 배경과 미래](http://news.pulmuone.kr/pulmuone/newsroom/viewNewsroom.do?id=1062)

[용어 레거시(legacy)란?](https://arabiannight.tistory.com/entry/IT%EC%9A%A9%EC%96%B4-%EB%A0%88%EA%B1%B0%EC%8B%9Clegacy-%EB%9E%80)


DVR은 Digital Video Recoder의 약자로 카메라에 잡히는 영상을 비디오 테이프없이 디지털화시켜 하드디스크에 압축 저장하는 영상저장 장치. 아날로그 시스템에서는 별도로 있어야하는 화면을 바꿔주는 스위치, 화면 분할기, VCR(비디오재생기), 센서 및 알람 제어기등을 하나로 통합한 시스템이지요.

이 DVR은 영상압축 알고리즘인 H.264,MPEG,MJPEG 등을 기반으로 한 기술로 동화상을 압축하고 저장합니다. 또 DVR은 디지털 이미지로 변화되어 녹화된 영상을 반영구적으로 HDD에 저장하는 기능과 사용자가 녹화된 데이타를 순간 검색할 수 있는 기능, 여러대의 카메라 영상을 1대의 모니터에서 분할하여 감시할 수 있는 멀티플렉서기능, 원격지에서도 lan,xdsl등에서 녹화검색 및 실시간 화면을 감시할 수 있는 화상전송기능이 내장되어 있습니다.



출처: [intelkorea](https://intelkorea.tistory.com/14)


# Metacogition (메타인지)

 '자신의 생각에 대해 판단하는 능력'을 말한다. ‘자기가 생각한 답이 맞는지’, ‘시험을 잘 쳤는지’, ‘어릴 때의 이 기억이 정확한지’, ‘이 언어를 배우기가 내게 어려울지’ 등의 질문에 답할 때에도 사용되며, 자신의 정신 상태, 곧 기억력이나 판단력이 정상인지를 결정하는 데에도 사용한다.  

 메타인지는 아이들의 발달 연구를 통해 나온 개념이므로 교육학 등에 주로 등장하는 용어다. 뛰어난 메타인지능력을 가졌다면 적절한 시기에 적절한 도전을 함으로써 학습속도를 빠르게 가져갈 수 있다. 예를 들어, 수영을 한달 배운 아이가 '나는 100m를 완주할 수 있는가'를 스스로 판단하고, 만약 완주할 수 없다면 나에게 부족한 게 체력인지 기술인지를 스스로 판단하는 데에 메타인지가 사용되므로 메타인지능력이 높다면 자신의 능력과 한계를 더욱 정확히 파악해 시간과 노력을 필요한 곳에 적절히 투자하므로 효율성이 높아진다.

 출처: [나무위키-메타인지](https://namu.wiki/w/%EB%A9%94%ED%83%80%EC%9D%B8%EC%A7%80)


 
# Assertion

 찾아보기

# 방어적 프로그래밍
일어난다고 가정하고 이에 대비하여 방지하는 것.

[방어적 프로그래밍 테크닉](https://ryudwig.tistory.com/entry/Code-Craft-Ch1-%EB%B0%A9%EC%96%B4%ED%95%98%EA%B8%B0)

[방어적 프로그래밍](http://statkclee.github.io/xwmooc-sc/novice/python/05-defensive.html)



# explicit 

C++ 11 이후에 자동 형변환이 된다고 한다.
처음 알았음..

쓰는 이유는 이해를 했지만, 조금 더 알아보고 정리해야 할 듯

[explicit은 뭘하는 건가요?](https://hashcode.co.kr/questions/325/c%EC%9D%98-explicit-%ED%82%A4%EC%9B%8C%EB%93%9C%EB%8A%94-%EB%AD%98-%ED%95%98%EB%8A%94-%EA%B1%B4%EA%B0%80%EC%9A%94)

[explicit 키워드](https://purestarman.tistory.com/110)


# stream

[H.264 stream 분석](http://egloos.zum.com/yajino/v/782492)

[기본 스트림](http://www.ktword.co.kr/abbr_view.php?m_temp1=3500)


# chmod 

이건 gst는 아닌데,  
그냥 한번 linux 관련해서 정리할 예정

[chmod 권한설정](https://conory.com/blog/19194)


# ctags

ctag란 프로그래밍 소스코드의 태그(전역변수 선언, 함수 정의, 매크로 선언)들의  database를 생성하는 Unix 명령어  

각 태그들의 인덱스를 만들어 편하게 찾을 수 있도록 만드는 유틸리티  

나중에 한번 깔끔하게 정리해야지.  
좋은 기능이군.  
[ctags](https://bowbowbow.tistory.com/15?category=176521)


# typedef 정의

구조체 정의를 할 때 쓰는 건 저번에 봤지만,  
함수를 정의하는 방식에 대해선 익숙하지 않아서 찾아봤다.

좀 더 example을 찾아보면서 나중에 공부해야겠다.  

[typedef 함수](https://dojang.io/mod/page/view.php?id=601)




# nohup python3 publish_pm_report.py > log.txt 2>&1 이해하기

[nohup 명령어란 무엇인가](https://klero.tistory.com/entry/nohup-%EB%AA%85%EB%A0%B9%EC%96%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80)

[쉘스크립트에서 이따금씩 사용되는 2>&1 이해하기](https://blogger.pe.kr/369)