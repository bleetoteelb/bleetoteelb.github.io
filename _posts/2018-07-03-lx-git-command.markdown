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

(source: https://comcxx11.github.io/com/add-remote-repository/)

.

.


# 2. How to make gitignore file?

_Gitignore_ file is for not to make public my credential information or not to upload unnecessary files.  
_Gitignore_ 파일은 공개하고 싶지 않은 개인 정보가 담긴 파일이나 업로드할 필요가 없는 파일들을 정리해두기 위해 만든다.

If you list up some files you want to except from add list,
they are automatically excepted.  
이 파일안에 파일 목록을 정리해두면 자동으로 add 리스트에서 빠진다.


__1__.. Make a _.gitignore_ file (Or you can set to be generated automatically when a repository is created in github. Find the option when you create repository)
```
$vi .gitignore
```
__2__.. list up the files according to the following rules

```
# a comment - this is ignored
*.a         # ignore .a files
!lib.a      # but include lib.a, even though it ignore .a files above
/abc.txt    # only ignore the root abc.txt files, not subdir/abc.txt
build/      # ignore all files in the build/ directory
doc/*.txt   # ignore doc/notes.txt, but not doc/sth/num.txt
```

(source: http://emflant.tistory.com/127)


.

.


# 2. Undo git add, commit or push

__1__.. Undo git add
```
$git reset
```

__2__.. Undo git commit
```
$git reset HEAD~1
```

__3__.. Undo git push
```
$git reset HEAD~1
$git push origin <branch> -f
```
