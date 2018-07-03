---
layout: post
title:  "내가 자주쓰는 Git 명령어"
subtitle:   "내가 자주쓰는 Git 명령어"
categories: devlog
tags: lx
---
내가 자주쓰는 Git 명령어

자주 쓰는 git 명령어 중에 
매번 쓰려 할 때마다 찾는 것들을 위주로 정리했다.

# 1. Add existed project to Github

__1__.. Create a new repository on Github. (Github에 새 repository를 만든다.)


__2__.. Change the directory to local project. (git bash같은 command창에서 현재작업폴더를 원하는 project 폴더로 바꾼다.)

__3__.. Initialize the local directory as a Git repository (해당 폴더를 초기화한다.)

```
$git init
```
__4__.. Add the files in new local repository and Commit the files (원하는 파일을 add하고 commit한다.)

```
$git add [ files you want to add ]
$git commit -m "[Your message]"
```

__5__.. Add remote repository (repository 주소를 등록한다.)
```
$git remote add origin [ github clone URL ]
```

__6__.. Push the changes to Github (현재까지의 상태를 Push한다.)