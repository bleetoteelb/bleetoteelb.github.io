---
layout: post
title:  "내가 자주쓰는 linux 명령어"
subtitle:   "내가 자주쓰는 linux 명령어"
categories: devlog
tags: lx
---
자주 쓰는 linux 명령어 중에 
매번 쓰려 할 때마다 찾는 것들을 위주로 정리했다.

딱히 특정한 분류는 없고
command창에서 쓰이는 모든 명렁어들을 업데이트 할 예정이다.

# 1. Add User

You can set basic information about the user including password when you create account.
Home directory is also automatically created.

계정 생성시 비밀번호까지 입력받으며 기본정보를 바로 입력 시켜줄 수 있다.
홈 디렉토리 또한 자동으로 생성된다.


```
$adduser [user name]
```

If not the root account, put _sudo_ in front of it.
root 계정이 아니라면 앞에 sudo 를 붙혀주자.

```
$sudo adduser [user name]
```

.

.


# 2. Change Password

```
$passwd [user name]
```

.

.

# 3. Delete user

```
$deluser [user name]
```

If you want to remove all files of user,

```
$deluser -remove-all-files [user name]
```


(source: http://mirwebma.tistory.com/112)