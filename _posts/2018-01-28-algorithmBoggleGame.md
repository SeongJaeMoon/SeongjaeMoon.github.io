---
layout: post
title:  "보글 게임 문제"
date:  2018-01-27 23:00:00
author: Seongjae Moon
categories: Algorithm
tags:   알고리즘 Java 재귀호출 완전탐색
---

요새 귀차니즘과 무기력증이 다시금 찾아와서, 만사가 귀찮아지는 기현상?이 발생했다. 이런걸 보고 흔히 슬럼프라고 하겠지.. 코딩은 해도해도 어렵게만 느껴지고, 아는 것 보다 알아야 할 것들이 훨씬 많은 이 분야에서, 내가 살아남을 수 있을까? 하는 막연한 걱정이 드는 요즘이다.

더군다나,  처음으로 겨울철 장염에 걸림으로 하여금 몸이 많이 약해졌음에 무서움을 또 한 번 느끼고, 건강이 최고라는 생각을 해본다.

넋두리는 이 쯤 하고, 어제 보다 오늘 더 뜻 깊은 날을 보내기 위해, 오늘도 알고리즘 문제를  풀어보도록 하자! ~~엥?~~ 2번 째 알고리즘 문제 포스팅! 보글 게임에 대한 포스팅 진행을 해보도록 하자.    

책에서는 C++를 이용해서 코딩하지만, 스터디를 위해 사용하기로 한 언어가 JAVA이기 때문에 코드는 JAVA로 작성되었으며, 책의 내용을 나름대로 재구성하여 작성하였다.

### 먼저, 보글 게임(Boggle Game)에 대해서 알아보자.
보글 게임은 알파벳 격자를 가지고 하는 게임으로, 상하좌우/대각선으로 인접한 칸들의 글자들을 이어서 단어를 찾아내는 게임이다.

중, 고등학교 시절 영어 시간에 이와 비슷한(거의 똑같은) 단어 찾기를 해본 기억이 새록새록 떠오른다. 그 땐 게임이라고 생각하진 않았다. ~~물론 지금도~~

### 예를 들어보자.
우선, 여기서 말하는 보글게임이란, 5X5 행렬에 들어있는 단어를 찾아내는 게임이다. 아래에 랜덤하게 정의된 대문자 알파벳이 있다고 가정하자.
![보글게임판](/assets/uploads/algorithm/Boggle1.jpg)
 여기서, 예를 들어 'TWICE', 'IS' ,'SO', 'BEAUTIFUL' 등의 단어를 찾는 다고 가정하면,

![단어찾기](/assets/uploads/algorithm/Boggle2.jpg)
위와 같이 알파벳 하나하나를 연결해서 원하는 값을 찾아낼 수 있다.

### 문제의 이해.
먼저, 단어를 찾아내기 위해선, 기준이 되는 위치에 인접한 칸을 하나씩 모두 찾아야 한다. 모든 인접한 칸을 비교해서, **한 칸이라도 연결 가능하다면 성공이다.**

![인접한 칸 찾기](/assets/uploads/algorithm/BoggleIndex.jpg)
위와 같이 **기준이 되는 (0,0)의 위치와 인접한 나머지 값을 하나하나 비교**해서, 찾는 알파벳이 있는지 확인하면 된다. 이제 코드를 살펴보자.
```java
//x값의 변화량을 가리킬 배열 선언
static final int [] dx = new int[]{-1, -1, -1,  1, 1, 1,  0, 0};
//y값의 변화량을 가리킬 배열 선언
static final int [] dy = new int[]{-1,  0,  1, -1, 0, 1, -1, 1};
//알파벳을 담을 2차원 배열 선언
static char[][] boggle = new char[10][10];

public static void main(String[] args) {
	int temp = -1;
	for(int x = 0; x < 5; ++x) {
		for(int y = 0; y < 5; ++y) {
			//배열에 A, B, C ... 순서대로 알파벳 대문자 할당
			char word = (char)(65+(++temp));
			boggle[x][y] = word;
		System.out.printf(String.format("%-2s ",boggle[x][y]));
		}
		System.out.println();
	}
	//찾는 단어
	String str = "MNOTYX";
	//결과 출력
	System.out.println(String.format("찾는단어:%s 결과:%s", str, isHasWord(str)));
}
/*
 기저사례
 1. 위치(y, x)에 있는 글자가 원하는 단어의 첫 글자가 아닌 경우 항상 실패
 2. (1번 경우에 해당되지 않을 경우) 원하는 단어가 한 글자인 경우 항상 성공
 */
public static boolean hasWord(int y, int x, String word) {
	//기저 사례 1: 시작 위치가 범위 밖이면 무조건 실패
	if(y > 5 || x > 5 || y < 0 || x < 0)return false;
	//기저 사례 2: 첫 글자가 일치하지 않으면 실패
	if(boggle[y][x] != word.charAt(0)) return false;
	//기저 사례3: 단어 길이가 1이면 성공
	if(word.length() ==1) return true;
	//인접한 여덟 칸을 검사한다.
	for(int direction = 0; direction < 8; ++direction) {
		//인접 y = 기준 y + y의 변화량, 인접 x = 기준 x + x의 변화량
		int nextY = y + dy[direction], nextX = x + dx[direction];
		//인접한 8개의 인덱스 중 일치하는 값이 있다면 (재귀호출)
		//다음 칸이 범위 안에 있는지, 첫 글자는 일치하는지 확인할 필요가 없다.
		if(hasWord(nextY, nextX, word.substring(1))) {
			//true 반환.
			return true;
		}
	}
	return false;
}
//5X5로 되어있는 2차원 배열의 시작 단어를 찾아내기 위한 메소드
public static boolean isHasWord(String word) {
	boolean result = false;
	int flag = 0;
	for(int i = 0; i < boggle.length; ++i) {
		for(int j = 0; j < boggle[i].length; ++j) {
			//찾는 단어의 첫 글자를 찾았다면
			if(boggle[i][j] == word.charAt(0)) {
				//인접한 알파벳 찾기 시작!
				result = hasWord(i, j, word);
				flag = 1;
				//반복 종료
				break;
			}
		}
		//중첩 반복문 종료
		if(flag == 1)break;
	}
	return result;
}

출력예:
A  B  C  D  E  
F  G  H  I  J  
K  L  M  N  O  
P  Q  R  S  T  
U  V  W  X  Y  
찾는단어:MNOTYX, 결과: true
```
위 코드의 순서를 간단하게 정리하면,
##### 1. 찾는 단어의 첫 번째 알파벳이 존재 하는지 확인.
##### 2. (1번을 만족하는 경우) 인접한 8칸을 비교해서, 찾은 알파벳 다음 알파벳이 있는지 비교.
##### 3. (2번을 만족하는 경우) 참을 반환하고, 모든 알파벳을 비교하기 까지 2번 반복.
위의 예에선 "MNOTYX"를 찾는 경우이므로, (2,2), (2,3), (2,4), (3,4), (4,4), (4,3)을 찾을 때까지 반복했다고 할 수 있다. 만약, 중간에 알파벳이 인접하지 않은(예를 들어, ABDE) 단어를 입력할 경우엔 false를 반환하게 된다.

### 시간 복잡도 분석.
완전 탐색 알고리즘의 시간 복잡도는 가능한 답 후보들을 모두 만들어 보기 때문에, 후보의 최대 수를 계산하면 된다. 알고리즘의 특성상 만약 찾는 알파벳이 없는 경우에도 모든 후보들을 빠짐 없이 검사하면서 단어의 끝에 도달해야 비로서 false를 반환 할 수 있다.

결과적으로, 단어의 길이 N에 대해 N-1 까지 진행하게 된다. 이 것을 빅-O 표기법으로 나타내면, **O(8^N)**으로 나타낼 수 있다.

그렇다. 이렇게 단어가 짧은 경우에는 재귀함수 호출을 통한 완전 탐색으로 해결 가능하지만,  글자가 길어지면 지수 시간의 알고리즘을 갖는 이 알고리즘으론 무리다. ~~무리데스네~~~  

여기서 중요한 부분은, **문제를 부분 문제로 쪼개서 각 단계 별로(재귀호출을 통해) 새로운 문제(부분 문제, 인접한 칸을 새롭게 비교)를 해결하는 방식이라는 점이다.**  완전 탐색을 대체할 다른 방법은 그 챕터가 나오면 풀어 보는걸로~

이렇게 해서 간단하게 Boggle Game에 대해 알아보았다. 책에선 난이도가 '하'로 적혀있는 나름 쉬운..? 알고리즘에 속하는 완전탐색 관련 알고리즘 문제이다. 역시 세상사 쉽지 않다.

계속해서 [다음](https://seongjaemoon.github.io/algorithm/2018/02/09/algorithmPicnic.html)은 소풍 문제를 풀어 보는걸로~

* 오타나 잘못된 부분을 지적 해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* 참고문헌<br>
구종만, 알고리즘 문제 해결 전략. pp.150~pp.155.
