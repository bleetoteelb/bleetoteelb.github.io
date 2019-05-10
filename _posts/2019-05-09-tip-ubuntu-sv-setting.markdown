---
layout: post
title: "Ubuntu에서 VScode 쓰기"
subtitle: "Ubuntu에서 VScode 쓰기"
categories: tip
tags: tip
comments: true
---

# __1. 개요__
Ubuntu에서 개발을 하지만,  
모든 작업을 vim editor로 하지는 않는다.  

지금 쓰고 있는 markdown과 같은 문서들은 preview를 봐야 편하기 때문이다.  
기타 IDE의 장점을 활용하고 싶은 사람들이 VScode 를 사용하기도 한다.   
( 마우스를 이용한 커서 이동, 함수 정의 따라가기, git UI 등 )  
<br>
<br>
# __2. 설치__

일단 설치는 두 가지 방법으로 할 수 있다.(Ubuntu LTS 18.04)
1. 홈페이지에서 다운받아서 설치
2. 명령어 입력으로 설치  
<br>

## 1. 홈페이지에서 설치

[https://code.visualstudio.com/](https://code.visualstudio.com/)  
VScode 홈페이지이다.

ubuntu에서 실행 가능한 버전인 .deb을 다운 받아서 실행하면 된다.  

다른 프로그램 설치와 다를 것이 없으므로 자세한 내용은 생략한다.  

<br>

## 2. 명령어 입력으로 설치

```bash
$ curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg

$ sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg

$ sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
```
 마이크로 소프트에서 microsoft.asc를 다운받아 microsoft.gpg로 저장한다.

 받은 GPG파일을 /etc/apt/trusted.gpg.d/로 옮긴다.

 apt의 소스 리스트에 추가한다.

```bash
$ sudo apt-get update
$ sudo apt-get install code
```

목록을 업데이트 한 후 VScode를 설치한다.

<br>
<br>  

# 3. 실행 및 세팅

VScode 내에서도 [__ctrl+`__] 버튼으로terminal을 쓸 수 있다.

Ubuntu상에서 기본으로 사용하는 쉘이 적용된다.  
bash, zsh 등등

Git이 이미 설치되어있다면 별도의 설정 없이 VScode terminal에서도 git을 사용할 수 있다.

![vsinit](https://bleetoteelb.github.io/assets/img/vsinit.png)  

빌드 관련 프로그램이 설치되어 있지 않다면,  
다음 명령어를 통해 기본적인 컴파일러, 라이브러리 등을 설치하자.  

```bash
sudo apt-get install build-essential
```

이후 extension 메뉴에 가서 C/C++과 Code Runner를 설치한다.  
(python, java, C#등등 본인이 사용하고자 하는 언어를 설치하면 된다.)  

![ccpp](https://bleetoteelb.github.io/assets/img/ccpp.png)  
![coderunner](https://bleetoteelb.github.io/assets/img/coderunner.png)  

간단한 c 파일을 만들어보았다.

![vsintell](https://bleetoteelb.github.io/assets/img/vsintell.png)  
IDE의 강력한 기능인 intelligence 기능  
<br>
![runcode](https://bleetoteelb.github.io/assets/img/runcode.png)  
코드를 실행하기 위해서는 __[ctrl+alt+n]__ 을 누르면 된다.  
오른쪽 클릭 -> Run Code 을 눌러도 된다.

실행파일은 c파일과 같은 폴더에 생성된다.
<br>  
<br>

# TIP

__1.__  
단축기 변경은   
File > Preferences > Keyboard Shortcut

![shortcut](https://bleetoteelb.github.io/assets/img/shortcut.png) 

__ctrl+alt+n__ 을 원하는 키로(다른것과 겹치지 않게) 바꿔도 된다.

<br>

__2.__  
Ubuntu에서 VScode를 틀면 한글이 이상하게 나온다.

'같은' 을 입력하면 '가ㅌ은' 이라고 나온다.

__File > Preferences > Setting  
Text Editor > font__  
에서

‘Droid Sans Mono’, ‘monospace’, monospace, ‘Droid Sans Fallback’

이 중 맨 뒤에 있는 
'Droid Sans Fallback'을 지워주면 된다.

짜잔!  
정상적으로 한글이 출력된다.

