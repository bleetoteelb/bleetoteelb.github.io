---
layout: post
title:  "Find words"
subtitle:   "Find words"
categories: devlog
tags: oop
---

# 1. 문제 

## 1-1. 설명

m x n 크기의 알파벳 행렬에서 원하는 단어를 가로, 세로, 대각선에서 찾기.  


## 1-2. 조건

1. 대소문자 구분 안 함
2. 1 <= m,n <=50
3. 찾아야 할 단어 갯수 k가 주어짐 (1 <= k <= 20)
4. 알파벳 이외의 것은 없음.


## 1-3. 입력
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

## 1-4. 출력
```
6 6
10 7
2 3
1 13
10 2
```

# 2. 해결전략

이 문제를 처음 받았을 때 보통 가장 먼저 드는 생각은  
행렬의 각 요소를 돌면서 8방향 탐색을 하는 방법일 것이다.  

나도 처음엔 그렇게 해결했다.  

그러나 이번엔 새로운 방법으로 구현해 보았다.  
행렬을 통째로 돌려서 string.find() 라는 아주 좋은 함수를 쓸것이다.  

이글에서는 두가지 방법을 모두 설명할 것이다.  

이글에서 찾을 단어를 word, 찾을 곳을 grid라고 부르겠다.  

## 2-1. 모두 탐색

이 방법은 로직 자체는 단순하다.  

![first](https://bleetoteelb.github.io/assets/img/first_method.JPG)

위 그림처럼 각 요소마다 8개 방향으로 찾는 word가 있는지 확인하면 된다.  

이를 위해 반복문을 겹겹히 써도 되고,  
자신이 있다면 recursive로 구현해도 된다.  

다만 이 방법은 단어의 철자를 확인하는 과정이 까다롭다.  
if문을 통해 여러가지 예외처리를 해주어야 한다.  

다음 진행해야 할 방향에 더 이상 철자가 없을때(가장자리 일때)와 같은 예외들이다.  

구현할 때 꽤나 골치를 썩인 기억이 있다.


## 2-2. Grid 회전


이 방법은 구현이 간단한 대신, 머리를 좀 써야한다.  
그 이유는 글을 읽다보면 알 것이다.  

### 2-2-1. Grids 만들기
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
![all](https://bleetoteelb.github.io/assets/img/all.JPG)



### 2-2-2. word 찾기

이제 이 grid들에서 word를 찾아보자.  
word를 grid에서 찾기 위해 string.find()함수를 이용할것이다.  

이 함수를 이용하기 위해서는 string header를 추가해야 한다.  
코드 맨 앞에 다음을 추가하자  
```{.cpp}
#include <string>
```

다음 find 함수 사용 예제를 보자.
```{.cpp}
#include <string>
#include <iostream>
using namespace std;

void print_result(string::size_type np){
	if(np != string::npos) cout << np << endl;
	else cout << "NOT FOUND" << endl;

	return;
}

int main(){

	string sentence = "Hello, my name is BleeToTeelB";
	string word1 = "name";
	string word2 = "github";

	string::size_type np;
	np = sentence.find(word1); // sentence string에서 word를 찾은 결과가 np에 저장될것이다.
	// 만약 찾지 못했다면 find함수는 npos가 반환한다.
	print_result(np);

	np = sentence.find(word2); 
	print_result(np);

	return 0;
}
```

결과는 다음과 같다.
```
10
NOT FOUND
```

이를 통해 우리는 반환되는 숫자는 counting이 배열과 같이 0부터 시작하고 띄어쓰기를 포함한다는 사실을 알 수 있다.  
물론 띄어쓰기는 지금 이문제에선 issue가 없는 사항이다.  




#### 2-2-3. 좌표 구하기

다시 4 x 7 grid로 돌아가보자.  

![i_color](https://bleetoteelb.github.io/assets/img/i_color.JPG)

위 그림에서 색칠된 i가 포함된 row를 string으로 만들어 find함수를 쓰면 반환값은 '4'가 될 것이다.  

이 함수를 이용하면 horizontal grid와 vertical grid는 금방 답을 구할 수 있다.  

문제는 회전된 grid이다.  
이들 grid에서 답을 구하기 위해서는 약간의 계산식을 이해할 수 있어야 한다.  

일단 다음 그림을 보자.

![i_color](https://bleetoteelb.github.io/assets/img/compare_number.JPG)

이제부터 column은 7이고 row는 4이다.

하늘색 화살표는 글자들의 숫자를 나타낸것이다.
각 grid에서 같은 색은 같은 글자들을 나타낸다.

이 중 초록색으로 칠해진 글자들은 지금부터 할 계산식의 가장 중요한 부분이다.
초록색으로 칠해진 곳은 grid의 column의 수보다 큰 곳이다.
화살표가 꺾이는 분기점이 이 부분으로 이에 따라 하얀색 부분과 초록색 부분의 계산식이 달라진다.

이제부터 노랜색으로 칠해진 g와 빨간색으로 칠해진 l의 좌표를 변환하는 식을 만들것이다.

##### 1. CLOCKWISE - 오른쪽 grid에서 i가 column보다 작은 부분 (i<column)


g는 오른쪽은 (4, 2)이고 왼쪽은 (3,3)이다.

행부터 확인해보면 이는 j와 관련이 있다.  
j가 1씩 커질수록 왼쪽 grid에서는 row가 커진다는것을 알 수 있다.  
따라서 (j+1, ??)이다.  

열은 i,j모두 관련이 있다.  
i가 1커질수록 열도 1씩 커진다.  
반대로 j가 1커질수록 열은 1씩 작아진다.  
따라서 (j+1, i-j+1)이다.  


##### 2. CLOCKWISE - 오른쪽 grid에서 i가 column보다 큰 부분 (i>=column)


이번엔 l을 살펴보자.  
l은 오른쪽은 (8,1), 왼쪽은 (4,6)이다.  

마찬가지로 행을 먼저 살펴보자.  
i가 1커질때마다 행도 1씩 커진다.  
하지만 column을 한번 더 빼줘야 한다.  
따라서 (i-column+j+2, ??)이다.  

이번엔 열을 살펴보자.  
열은 j와만 관련이 있다.  
j가 1 커질때마다 열은 1씩 줄어든다.  
따라서 (i-column+j+2, column-j)이다.  

같은 방법으로 반시계 방향으로 돌린 grid의 식도 계산해보자.  
설명은 생략하겠다.  

##### 3. COUNTER-CLOCKWISE - 오른쪽 grid에서 i가 column보다 작은 부분 (i<column)

(j+1, column-i+j)


##### 4. COUNTER-CLOCKWISE - 오른쪽 grid에서 i가 column보다 큰 부분 (i>=column)

(i-column+j+2, j+1)


#### 2-2-4. 뒤집어진 경우

맨 처음에 소개한 한 점을 중심으로 8방향을 탐색하는 방법과 비교해 봤을 때,

우린 아직 4방향밖에 확인하지 않았다.  

나머지 4방향은 word를 뒤집어서 같은 find를 하면 된다.  
이때 find의 반환값은 word의 끝 글자일것이다.  

즉 korea를 뒤집어서 aerok를 찾으면  
find함수는 a의 위치를 반환할 것이다.  

이는 위에 계산한 계산식에서  
화살표 방향을 보고 word의 길이를 더하거나 빼면 구할 수 있다.  


### 2-3. 결과 확인

![test_result](https://bleetoteelb.github.io/assets/img/test_result.JPG)


###### 소스는 공개하지 않습니다.

## Tip

1. 대소문자가 상관없으니 처음에 전부 대문자 혹은 전부 소문자로 미리 바꾸면 편하다.
2. 계산식이 이해가 되지 않는다면 직접 그려서 확인해보기를 추천한다.
3. 회전된 grid의 0열의 좌표를 먼저 확인하고 j가 커질때마다 기존 grid에서 어디로 이동하는지 확인하면 좀 더 쉽다.
4. string을 뒤집는 함수는 reverse()를 쓰면 된다. 헤더로 algorithm을 추가해주자.  

```{.cpp}
#include <iostream>
#include <algorithm>
using namespace std;

int main(){

    string word = "korea";
    string reversed_word = word;

    reverse(reversed_word.begin(),reversed_word.end());
    cout << "Original word is " << word << endl;
    cout << "Reversed word is " << reversed_word << endl;
}

```

그 결과  

```
Original word is korea
Reversed word is aerok
```


# 3. 추가 테스트 예제

다음 여러가지 테스트 셋을 더 만들어보았다.  
본인의 코드를 테스트해보길 바란다.  

##### 예제1
---

```
1

8 11
abcDEFGhigg
hEbkWalDork
FtyAwaldORm
FtsimrLqsrc
byoArBeDeyv
Klcbqwikomk
strEBGadhrb
yUiqlxcnBjf
4
Waldorf
Bambi
Betty
Dagbert
```

결과
```
2 5
2 3
1 2
7 8
```

##### 예제2
---

```
1

10 15
eCiWtERYfdgvXVb
CVwQWPqdahjAdfU
CnfeReQxAjhpcgH
zcNlQWNHtYpsxoe
vonggpeEReradoo
doroTuyeedLakgn
CDgoQerFuucipeg
CiBgcxQQQQQsdlr
qyuhJhgffghrrtw
cuYtyioFGHeaads
6
QQQQQs
goOGle
QueeN
UhEong
Twice
Gogi
```

결과

```
8 7
8 4
8 11
2 15
1 5
5 5
```

##### 예제3
---

```
1

20 10
werydSFhku
DesfghdSKe
XCWasDSWwP
CVbsDasdae
cASgaserer
sbdaGerwnq
CDuAGsergy
dfskFvcbgh
erStbryuwt
Qwtqwusfad
eruyiekEli
tyiggyubii
WPAWOEJaus
aERYJOwetk
ueYwqEYwRt
TgOyuhjmgn
cSghsdcisg
wqenDFfynp
retUariist
rtolgjjZsp
4
Janggu
bukbukbuk
Kkwaenggwali
Jing
```

결과

```
20 6
6 2
1 9
18 7
```