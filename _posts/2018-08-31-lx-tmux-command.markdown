---
layout: post
title:  "tmux 명령어"
subtitle:   "tmux 명령어"
categories: devlog
tags: lx
---
내가 자주쓰는 Tmux 명령어

난 IDE보다 UNIX 환경에서의 코딩을 좋아한다.  
나한테 더 편하다거나 내가 그게 더 빠르다거나 그렇다기보단,  
__그냥 기분이 그게 더 좋다.__

Ubuntu 서버를 만든 이유 중 하나도 그 때문인데,
Ubuntu를 ssh로 접속하면 tmux로 창분할을 해서 코딩을 하게 된다.

그럼에도 가끔가다 까먹는 명령어가 있어서 생간난 김에 정리해두려 한다.

.  


# 1. Tmux란?

tmux는 터미널 화면을 여러개로 분할하고,  
세션을 저장하고(attach), 나중에 저장된 세션을 다시 불러올 수 있는(detach) 편리한 기능을 갖고있다. 


즉, 매번 SSH 연결을 할때마다 해당 directory에 일일히 들어갈 필요 없이  
이전에 작업하던 창 그대로 이어서 할 수 있다는 뜻.

 
 .  
# 2. 설치방법

Ubuntu 기준으로  

```
$ apt-get install tmux
```

만약 root 계정이 아니라면 맨 앞에 sudo를 붙혀줘야 한다.

.  

# 3. 명령어들

tmux는 __ctrl + b__ 가 기본이다.  

즉,
```
ctrl + b, <key>
```
차례대로 누르면 된다.

예를들어, 횡분할을 하는 key는 %인데,  
__ctrl + b__ 와 __%__ 를 차례로 누르면 된다.  

.  


### 3-1. tmux 모드로 들어가기 전 명령어

```
# 새 세션 생성
$ tmux // 이것만 치면 auto-increment 숫자만 가진 상태로 만들어진다.
$ tmux new -s <session-name>

# 세션 이름 수정
ctrl + b, &

# 세션 목록 보기
$ tmux ls

# 저장된 세션 다시 시작
$ tmux attach -t <session-number or session-name>
```

.  

### 3-2. tmux 모드 들어가서 세션 관련


```
# 세션 이름 수정
ctrl + b, $

# 세션 종료
$ exit
ctrl + b, x // 틀&윈도우가 한 개일때만 세션 종료
ctrl + d    // 틀&윈도우가 한 개일때만 세션 종료

# 세션 나가면서 저장하기 (detach)
ctrl + b, d
```

.  

### 3-3. tmux 윈도우 관련
```
# 새 윈도우 생성
ctrl + b, c

# 윈도우 이름 수정
ctrl + b, ,

# 윈도우 종료
$ exit // 틀이 한 개일때만 윈도우 종료
ctrl + b, &
ctrl + d

# 윈도우 이동
ctrl + b, 0-9 // window number 
```
.  

### 3-4 틀 관련
```
ctrl + b, % // 횡 분할
ctrl + b, " // 종 분할

# 틀 이동
ctrl + b, <방향키>

# 틀 삭제
$ exit
ctrl + b, x
ctrl + d

# 틀 사이즈 조절
(ctrl + b, :)
resize-pane -L 10
            -R 10
            -D 10
            -U 10

# 틀 레이아웃 변경
ctrl + b, spacebar

# 틀 하나를 윈도우로 확장하기
ctrl + b, z // 해제도 같은 명령어
```


