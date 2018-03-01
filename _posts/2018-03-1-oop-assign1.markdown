---
layout: post
title:  "Find words"
subtitle:   "Find words"
categories: devlog
tags: oop
---

# 1. 문제 

## 설명

m x n 크기의 알파벳 행렬에서 원하는 단어를 가로, 세로, 대각선에서 찾기.


## 조건

1. 대소문자 구분 안 함
2. 1 <= m,n <=50
3. 찾아야 할 단어 갯수 k가 주어짐 (1 <= k <= 20)
4. 알파벳 이외의 것은 없음.


## 입력
```
1

10 13
wAWasDaweTdwK
dEOYffcJTugot
VCbBRUIRdfrgq
aWEWJdfypEwex
wyeereDGApdsq
dcBVCNCsrfgfg
wiQPWadtvPdsP
aeWeyVvksdoae
GoKbnewqqWqts
ctVsdrgITHubw
5
Naver
Github
Object
Korea
Toeic
```

## 출력
```
6 6
10 7
2 3
1 13
10 2
```

# 해결전략

이 문제를 처음 받았을 때 보통 가장 먼저 드는 생각은
행렬의 각 요소를 돌면서 8방향 탐색을 하는 방법일 것이다.

나도 처음엔 그렇게 해결했다.

그러나 이번엔 새로운 방법으로 구현해 보았다.
행렬을 통째로 돌려서 string.find() 라는 아주 좋은 함수를 쓸것이다.

이글에서는 두가지 방법을 모두 설명할 것이다.

이글에서 찾을 단어를 word, 찾을 곳을 grid라고 부르겠다.

### 1. 모두 탐색

### 2. Grid 회전

이 방법은 구현이 간단한 대신, 머리를 좀 써야한다.
그 이유는 글을 읽다보면 알 것이다.

첫번째 단계는 grid 4개를 만드는 것이다.
> 1. Horizontal grid
> 2. Vertical grid
> 3. Clockwise rotated grid
> 4. Counter-clockwise rotated grid

설명의 편의를 위해 예제보다 작은 grid를 새로 만들겠다.

```
abcdefg
cdefghi
efghijk
ghijklm
```
4 x 7 grid이다.

이를 방향에 맞게 돌려보면 다음과 같은 grid들이 나온다
![all](/img/all.JPG)

## Tip

1. 대소문자가 상관없으니 처음에 전부 대문자 혹은 전부 소문자로 미리 바꾸면 편하다.