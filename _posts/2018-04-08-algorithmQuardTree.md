---
layout: post
title:  "쿼드 트리 뒤집기 문제"
date:  2018-03-11 23:00:00
author: Seongjae Moon
categories: Algorithm
tags:   분할정복 알고리즘 Java 쿼드트리뒤집기
---
오랜만에 알고리즘 포스팅~ 이번 포스팅에선 분할 정복 알고리즘 중 쿼드 트리 뒤집기라는 재밌는? 이름을 가진 문제에 대한 정리를 진행해보자.

책에서는 C++를 이용해서 코딩하지만, 스터디를 위해 사용하기로 한 언어가 JAVA이기 때문에 코드는 JAVA로 작성되었으며, 책의 내용을 나름대로 재구성하여 작성하였다.

### 먼저, 쿼드 트리 뒤집기란 무엇인지 알아보자.
쿼드 트리란 대량의 좌표 데이터를 메모리 안에 압축해 저장하기 위해 사용하는 여러 기법 중 하나이다. 주어진 공간을 항상 4개로 분할해 재귀적으로 표현하기 때문에 쿼드 트리라는 이름이 붙었으며, 이의 유명한 사용처 중 하나는 검은 색과 흰 색밖에 없는 흑백 그림을 압축해 표현하는 것이다.

### 예를 들어보자.
쿼드 트리는 $$2^n \cdot 2^n$$ 크기의 흑백 그림을 아래와 같은 과정을 거쳐 문자열로 압축한다.
1. 그림의 모든 픽셀이 검은 색일 경우 이 그림의 쿼드 트리 압축 결과는 그림의 크기에 관계없이 b가 된다.
2. 그림의 모든 픽셀이 흰 색일 경우 이 그림의 쿼드 트리 압축 결과는 그림의 크기에 관계없이 w가 된다.
3. 모든 픽셀이 같은 색이 아니라면, 쿼드 트리는 이 그림을 가로 세로로 각각 2 등분해 4개의 조각으로 쪼갠 뒤 각각을 쿼드 트리 압축한다. 이때 전체 그림의 압축 결과는 x(왼쪽 위 부분의 압축 결과)(오른쪽 위 부분의 압축 결과)(왼쪽 아래 부분의 압축 결과)(오른쪽 아래 부분의 압축 결과)가 된다.

![쿼드트리](/assets/uploads/algorithm/quardtree.png)
위 그림과 같이 16x16크기의 예제 그림은 쿼드 트리가 어떻게 분할해 압축하는지를 보여준다. 이때 전체 그림의 압축 결과는 xxwww bxwxw bbbww xxxww bbbww wwbb가 된다.

쿼드 트리로 압축된 흑백 그림이 주어졌을때, 이 그림을 상하로 뒤집은 그림을 쿼드 트리 압축해서 출력하는 프로그램을 작성하는 것이 목표인 문제다.

- 시간 및 메로리 제한
프로그램은 1초 내에 실행 되어야 하고, 64MB 이하의 메모리만 사용해야 한다.
- 입력
입력의 첫 줄에는 테스트 케이스의 수 C(C<=50)가 주어진다. 그 후 C줄에 하나씩 쿼드 트리로 압축한 그림이 주어진다. 모든 문자열의 길이는 1,000 이하이며, 원본 그림의 크기는 $$2^{20} \cdot 2^{20}$$을 넘지 않는다.
- 출력
각 테스트 케이스당 한 줄에 주어진 그림을 상하로 뒤집은 결과를 쿼드 트리 압축해서 출력한다.
```
입력예:
w
xbwwb
xbwxwbbwb
xxwwwbxwxwbbbwwxxxwwbbbwwwwbb
출력예:
w
xwbbw
xxbwwbbw
xxwbxwwxbbwwbwbxwbwwxwwwxbbwb
```
### 문제의 이해.
이 문제를 풀 수 있는 가장 무식한 방법은 주어진 그림의 쿼드 트리 압축을 풀어서 실제 이미지를 얻고 상하 반전한 뒤 다시 쿼드 트리 압축하는 것이다. 문제에 주어진 원본 그림의 크기 제한을 보면 이러한 방법으로 구현할 수 없다는 것을 알 수 있다.

#### 작은 입력에 대해서만 동작하는 단순한 알고리즘으로부터 시작해서 최적화 해 나가기.

우선, 기저 사례는 s의 첫 글자가 w나 b인 경우이고, 이때는 배열 전체에 해당색을 칠하고 곧장 종료한다. 만약 첫 글자가 x라면 decompress()는 s의 나머지 부분을 넷으로 쪼개 재귀 호출한다. 이때 각 부분이 배열의 어느 부분에 저장되어야 하는지 지정하는 위치 오프셋 또한 전달한다. 다음과 같은 형의 decompress()를 작성하면 된다.
```java
String decompress[max_size][max_size];
//w를 압축 해제해서 decompress[y..y + size - 1][x..x + size - 1]구간에 쓴다.
String decompress(final String s, int y, int x, int size){

}
```

- 압축 문자열 분할하기

이 상태에서 구현을 시작하면 s의 나머지 부분을 넷을 쪼개기가 까다롭다. 이런 때 유용하게 쓸 수 있는 패턴은 s를 미리 쪼개는 것이 아니라 decompress 함수에서 **필요한 만큼 가져다 쓰도록** 하는 것이다. decompress 함수에 s를 통째로 전달하는 것이 아니라, s의 한글자를 가리키는 값을 전달한다. 함수 내에서는 이터레이터를 구현해서 한 글자를 검사할 때마다 가리키는 값을 한 칸씩 옮긴다. 재귀 호출 외부에서 pointer 라는 static 변수를 선언하여 반복하는 역할을 할 임의의 변수를 선언한다. 포인터 역할을 하는 이 변수는 재귀 호출이 진행 될 때 마다 값이 하나씩 증가한다. 따라서 decompress 함수가 종료하고 나면 해당 변수는 항상 다음 부분의 시작 위치를 가리키게 된다.
```java
String decompress[max_size][max_size];
static int pointer;
void decompress(int pointer, int y, int x, int size){
  //한 글자를 검사할 때마다 포인터 역할을 하는 변수를 한 칸 앞으로 옮긴다.
  char head = quardtree.charAt(pointer);
  pointer++;
  //기저 사례: 첫 글자가 b 또는 w인 경우
  if(char == 'b') || char == 'w'){
    for(int dy = 0; dy < size; ++dy){
      for(int dx = 0; dx < size; ++dx){
        decompress[y + dx][x + dx] = head;
      }
    }
  }else{
    int half = size / 2;
    decompress(iter, y, x, half);
    decompress(iter, y, x+half, half);
    decompress(iter, y+half, x, half);
    decompress(iter, y+half, x+half, half);
  }
}
```

- 압축 다 풀지 않고 뒤집기

압축을 푸는 메서드를 작성했다고 해서 이 문제를 다 푼 것은 아니다. 곧이 곧대로 압축을 풀기에는 이 문제에서 다루는 그림들은 너무 크기 때문이다. $$2^{20} \cdot 2^{20}$$ 크기의 거대한 그림 맨 왼쪽 위에 검은 픽셀이 하나 있고 나머지는 전부 흰 픽셀이라고 가정해보자. 쿼드 트리는 이런 그림을 아주 짧게 압축할 수 있지만 압축을 해제한 결과는 1테라바이트나 되기 때문에 우리가 다룰 수 없는 양이다.

이러한 문제를 해결하기 위해 그림을 절반씩 나눠 뒤집는 방식을 취해야 한다. 원본 그림을 4등분해서 각 부분을 1, 2, 3, 4로 부르기로 했다고 가정하자. 이때 재귀 호출을 이용해 네 부분을 각각 상하로 뒤집은 압축 결과를 얻었다면, 각 부분을 합치면 각 부분이 상하로 뒤집힌 그림이 된다. 여기에서 위 두 조각과 아래 두 조각을 각각 바꾸면 이것이 애초에 우리가 원하던 전체가 뒤집힌 그림이 된다.

이 아이디어에 기초해 아래 코드를 작성하면 위 코드와 거의 비슷한 구조를 가지고 있지만, 오히려 훨씬 간단한 코드를 작성할 수 있다.
```java
package com.test;

import java.util.*;

public class QuardTree {

	static int pointer;
	static String qt;

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		String ret = "";
		System.out.print("테스트 케이스(T<50)");

		int t = sc.nextInt();
		sc.nextLine();

		for(int i = 0; i < t; ++i) {
			System.out.print("값:");
			pointer = 0;
			qt = sc.nextLine();
			if(qt.length() > 1000)continue;
			ret += reverse() + "\n";
		}

		System.out.print(ret);

		sc.close();
	}

	public static String reverse() {
		//처음 글자가 x가 아니라면
		if(qt.charAt(pointer) != 'x') {
			pointer++;
			return qt.charAt(pointer - 1) + ""; //String 형으로 리턴하기 위해 빈 문자열을 더하기.
		}else {//처음 글자가 b이거나 w 일 경우
			pointer++;
			//왼쪽 위.
			String ul = reverse();
			//오른쪽 위.
			String ur = reverse();
			//왼쪽 아래.
			String ll = reverse();
			//오른쪽 아래.
			String lr = reverse();

			//순서대로 뒤집기 -> 왼쪽 아래, 오른쪽 아래, 왼쪽 위, 오른쪽 위  
		return 'x' + ll + lr + ul + ur;
		}
	}

}
/*
출력예:
테스트 케이스(T<50)3
값:w
값:xbwwb
값:xbwxwbbwb
w
xwbbw
xxbwwbbbw
*/
```
위 코드의 순서를 간단하게 정리하면,
##### 1. 입력받은 문자열의 첫항의 값을 찾고, 기저 사례를 구분한다.
##### 2. 왼쪽, 오른쪽 위 아래를 각 부분으로 재귀 호출로 나눠서 구한다.
##### 3. 2번을 만족하는 경우 위 두 조각과 아래 두 조각을 각각 바꿔가며 답을 구한다.

### 시간복잡도 분석.
위의 코드에서 reverse 함수는 한 번 호출될 때마다 주어진 문자열의 한 글자씩을 사용한다. 따라서 함수가 호출되는 횟수는 문자열의 길이에 비례하므로 빅-오 표기법으로 O(n)이 된다. 각 문자열을 합치는데 O(n)의 시간이 든다 해도, 시간 안에 충분히 수행할 수 있는 시간이다.<br>

이렇게 해서 책에 역시나 난이도 '하'로 나와 있는 어려운? 쿼드 트리 뒤집기 문제를 정리했다. 재귀 호출로 해결할 수 있는 부분이 생각보다 굉장히 많다는 것을 또 새삼 느끼고, 재귀재귀재귀를 외치며 오늘 하루를 마무리 해야겠다. 다음 포스팅에선 계속해서 울타리 잘라내기 문제를 정리해보는 걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* 참고문헌<br>
구종만, 알고리즘 문제 해결 전략. pp.189~pp.195.
