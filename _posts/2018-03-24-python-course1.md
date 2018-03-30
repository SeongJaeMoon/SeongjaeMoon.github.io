---
layout: post
title:  "파이썬 기본 자료형 및 문법 정리"
date:   2018-03-24 17:32:00
author: Seongjae Moon
categories: Python
tags:   Python 파이썬 자료형 문법 Atom Java
---

오랜만에 블로그 포스팅! 개인적인 공부와 스터디를 위해 파이썬 기초부터 웹 크롤링에 대한 정리를 진행하게 되었다. 그 첫 번째로 파이썬의 기초 문법과 자료형에 대한 정리를 시작해보자. 엄격한? 언어인 Java 코드에 익숙한 나를 비롯한 여러 개발자 분들을 위해 비교해가면서 함께 작성했다. 테스트 코딩용 파이썬 에디터는 [Atom](https://atom.io/)과 script 패키지를 이용해서 작성하였고, [Python 3.6.4](https://www.python.org/downloads/) 버전을 사용했다. Java 코드 테스트는 마찬가지로 [Atom](https://atom.io/)과 JDK 8 환경에서 코딩되었다. 본 포스팅의 내용은 개인적인 공부와 스터디를 위해 기초적인 부분이 많이 포함되어 있으므로, 고급 개발자 분들께선 넘어가셔도 좋을 것 같습니다.

# Life is too short, you need python.
우선 파이썬은 네덜란드의 **천재** 개발자 귀도 반 로섬이 개발을 시작한 언어로 인터프리터 모드로 동작한다. 직관적이고 가독성 좋은 키워드들과 많은 양의 표준 라이브러리 지원 및 여러 갓 개발자들이 개발한 다양한 서드 파티 라이브러리 덕분에 프로그래밍 언어를 처음 배우는 입문자에게 많이 권하는 프로그래밍 언어라고 할 수 있다. ~~난 왜 대학에서 배우지 않았는가..~~ 그렇다면 거두절미하고, 왜 그렇게 강력하고 빠르게 배울 수 있다고 표현하는지 정리해보도록 하자.

## 파이썬 설치 및 아톰 에디터 준비
<!-- ![파이썬 다운로드 홈페이지](/assets/uploads/python/python1/pythondownload.png) -->
우선, [여기서](https://www.python.org/downloads/) 파이썬 3 버전을 OS에 맞게 다운로드 받는다. 파이썬을 다운로드 받고나면 IDLE를 비롯한 여러가지 파일들이 함께 설치된다. 만약 IDLE가 아닌 다른 에디터에서 파이썬 코딩을 하기 위해선 환경 변수를 설정 해주어야 한다.

- Windows

탐색기 창을 열고 시스템 환경 변수 설정을 실행한다. 시스템 환경 변수 창에서 Path를 클릭한다. Path를 설정하는 창이 나오면 파이썬 설치 위치를 경로에 추가한다. (윈도우 7을 사용하는 사용자는 Path 부분 마지막에 ;를 입력한다.)

만약 파이썬이 설치된 경로를 확인하기가 어렵다면, 아래처럼 IDLE를 실행하고 sys.path를 입력하면 파이썬의 설치 경로를 확인할 수 있다. 여기선, C:\\Users\\SIST\\AppData\\Local\\Programs\\Python\\Python36-32가 Path에 설정해야 할 경로라고 할 수 있다.
```
Python 3.6.4 (v3.6.4:d48eceb, Dec 19 2017, 06:04:45) [MSC v.1900 32 bit (Intel)] on win32
Type "copyright", "credits" or "license()" for more information.
>>> import sys
>>> sys.path
['', 'C:\\Users\\SIST\\AppData\\Local\\Programs\\Python\\Python36-32\\Lib\\idlelib',
'C:\\Users\\SIST\\AppData\\Local\\Programs\\Python\\Python36-32\\python36.zip',
'C:\\Users\\SIST\\AppData\\Local\\Programs\\Python\\Python36-32\\DLLs',
'C:\\Users\\SIST\\AppData\\Local\\Programs\\Python\\Python36-32\\lib',
'C:\\Users\\SIST\\AppData\\Local\\Programs\\Python\\Python36-32',
'C:\\Users\\SIST\\AppData\\Local\\Programs\\Python\\Python36-32\\lib\\site-packages']
>>>
```

- Mac OS

파이썬은 C 언어를 기반으로 만들어졌기 때문에, C 언어 컴파일러가 필요한데 Xcode나 OSX-GCC-Installer 둘 중 하나를 선택해서 다운로드한다. ~~ios 개발도 할 겸 Xcode를 앱 스토어에서 다운로드하자..~~

맥 OS에는 기본적으로 python 2.7이 설치되어 있는데 우리는 파이썬 3을 이용할 것이므로 파이썬 3을 설치해야 한다. 설치 방법은 [여기](http://python-guide-kr.readthedocs.io/ko/latest/starting/install/osx.html)를 참고. brew를 이용하거나 dmg 파일을 이용해 파이썬 3을 설치했다면 터미널을 열고 python3을 입력해보자. 아래처럼 나오면 정상적으로 파이썬 3가 설치된 것이다.
```
Python 3.6.4 (v3.6.4:d48ecebad5, Dec 18 2017, 21:07:28)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>exit()
```
atom 에디터를 사용할 때 파이썬 2는 인코딩 문제 등 귀찮은 작업들이 여럿 필요하다. 경로 설정이 정상적으로 export 되었다면 잘 실행되지만, 간혹 가다 아톰 에디터 설정이 파이썬 3으로 되어있지 않은 경우가 있다. 그럴 땐 Install a package를 실행하고 Settings 창이 나오면 Install 버튼 밑에 Open Config Folder를 찾아서 누른다.

![python3 설정](/assets/uploads/python/python1/python3setting.png)
Open Config Folder를 클릭하면 새로운 창이 열리면서 프로젝트 트리 뷰에 .atom 이라는 프로젝트가 추가된다. 이때 .atom 프로젝트를 열고 packages 밑에 script창을 연다. 새로운 창이 열리면 lib > grammars > python.coffee를 순서대로 실행한다. 위 사진처럼 exports.Python = 코드 command 옆에 'python'을 'python3'로 수정한다. 정상적으로 수정되었는지 확인하기 위한 가장 간단한 방법은 파이썬 코드에 print "123"을 입력하고 실행했을 때 에러가 발생하면 정상적으로 수정된 것이다.

아톰 에디터에서 python 3.x 버전을 사용할 때 print 함수에 한글을 입력하면 에러가 발생하는 현상을 마주하게 되는데, 아래 구문을 코드 최상단부에 작성하면 이를 해결할 수 있다.
```python
import sys
import io
sys.stdout = io.TextIOWrapper(sys.stdout.detach(), encoding = 'utf-8')
sys.stderr = io.TextIOWrapper(sys.stderr.detach(), encoding = 'utf-8')

```
- Linux Ubuntu

리눅스는 Add/Remove 관리자를 실행해서 파이썬을 찾아서 다운로드해주면 된다. 설치와 관련된 더 자세한 내용은 [여기](https://juehan.github.io/DiveIntoPython3_Korean_Translation/installing-python.html#macosx)를 참고하자.

<!-- ![아톰 다운로드](/assets/uploads/python/python1/atomdownload.png) -->
- 파이썬 코딩을 원할하게 진행하기 위해 Atom [사이트](https://atom.io/)에서 아톰 에디터를 다운로드 받는다.

<!-- ![아톰에서 사용할 파이썬 개발용 패키지 다운로드](/assets/uploads/python/python1/scriptdownload.png) -->
- 아톰 에디터를 다운로드 받았다면, Install a Package 에서 script와 autocomplete-python 패키지를 다운로드 받는다.
```python
print("hello world")
```
새로운 파이썬 파일을 만들고(.py 확장자를 가진 파일) 저장한다. 저장 후, 위 코드를 작성하고 실행했을 때 스크립트가 정상적으로 실행되면 성공이다. 만약 실행 코드 화면이 너무 작아서 눈이 아프다면 File > StyleSheet...를 클릭하고 아래 코드를 붙여넣는다.
```
.script-view .line{
	font-size: 17px;
}
```

### 파이썬 기초 문법
파이썬의 기초 문법에 대해 알아보기 전에 알아두면 좋은 코드 스타일 내용에 대해 알아보자. 단, 아래 코딩 스타일에 내용 중 인스턴스라던지 속성과 같은 내용은 파이썬 기초 문법 수준을 넘어서니 차차 정리하는 걸로 하고 , 나중에 내가 짠 코드의 가독성과 더불어 다른 개발자가 짠 코드를 빠르게 해석하기 위한 내용 정도로 생각하면 될 것 같다.


종류 | 스타일
:-|:-:|
코드블록|파이썬은 타 언어(Java, C...)와 같이 중괄호 {}를 이용하여 코드 블록을 구분하지 않는다. 탭이나 스페이스 바를 통해 코드를 구분 짓는다.(스페이스 바 4번을 권장.) 또한, if, for, while, def문과 같이 특정 키워드 다음에 나오는 문장은 콜론 :를 통해 구분 짓는다. 점차 익숙해지고 나면 괜찮지만 처음엔 많이 어색해서 자동적으로 자바 문법을 작성하게 된다. ~~내가 그렇다~~ 에디터를 이용하는 큰 이유이기도 하다.
라인구분|파이썬에서 선언한 함수나 클래스는 2개의 공백 라인을 추가하여 구분한다. 메서드는 한 개의 공백 라인으로 구분한다.
인덱싱|컬렉션의 인덱스나 함수 호출, 함수 파라미터 등에서 불필요한 공백을 넣지 않는다
명명규칙| 함수, 변수, Attribute는 소문자로 단어 간은 밑줄을 사용하여 연결한다.(스네이크 케이스)
명명규칙|클래스의 protected instance attribute는 하나의 밑줄로 시작한다
명명규칙|클래스의 private instance attribute는 2개의 밑줄로 시작한다
명명규칙|인스턴스 메서드는 (객체 자신을 가리키기 위해) self 를 사용한다.
명명규칙|클래스 메서드는 (클래스 자신을 가리키기 위해) cls 를 사용한다
문장과표현식| if, for, while 블록 문장은 한 줄에 작성하지 않는다.
문장과표현식|a는 b가 아니다를 표현할 때 a is not b 를 사용한다. not a is b 를 사용하지 말 것
문장과표현식|값이 비어있는지 아닌지를 검사하기 위해 길이를 체크하는 방식을 사용하지 말 것. 대신 if mylist 와 같이 표현함


코딩 스타일에 관련된 더 자세한 내용은 [여기](http://pythonstudy.xyz/python/article/511-%ED%8C%8C%EC%9D%B4%EC%8D%AC-%EC%BD%94%EB%94%A9-%EC%8A%A4%ED%83%80%EC%9D%BC)와 [여기](http://python-guide-kr.readthedocs.io/ko/latest/writing/style.html)에 아주 잘 나와있다. 다음은 파이썬의 자료형에 대해서 알아보도록 하자.

### 숫자 자료형 &amp; 논리 자료형


타입|설명|예
:-:|:-:|:-:
int|정수형 -21억 ~ +21 억 (32bit)|100, 0xFF(16), 0o56(8진수)
long int| -922경 ~ +922경 (64bit) (python 2에서만 동작)| a = 100L
float|소숫점을 포함한 수|a = 10.01
bool|참, 거짓을 표현하는 불 값|a = True <br> a = False
None|Null 값|a = None
# | 한 줄 주석으로 샵 기호를 쓰고 한 칸 띄우는 것을 권장한다고 한다.| # 파이썬
'''<br>''' | 다중 라인 주석 | '''<br>파이썬은 정말이지 멋진 언어이다.<br> '''


```python
int(3.5)      # 3
2e3           # 2000.0
float("1.6")  # 1.6
float("inf")  # 무한대
float("-inf") # -무한대
bool(0)       # False. 숫자에서 0만 False
bool(-1)      # True
bool("False") # True
a = None      # a는 None
a is None     # a가 None 이므로 True
```
- 위 코드는 파이썬에서 사용하는 기본적인 숫자 자료형과 논리 자료형에 대한 표현이다. 대소문자를 구분하며, 파이썬은 타입 추론을 지원하기 때문에 동적으로 타입이 정해진다. 때문에 Java와 같이 변수 선언 전에 타입을 선언할 필요가 없다. 또한, 타 언어에선 보기 힘든 None이라는 타입이 있는데, null 값 이라고 생각하면 간단하게 이해가 가능하다. 더 자세한 내용은 [여기](http://pythonstudy.xyz/python/article/7-%EA%B8%B0%EB%B3%B8-%EB%8D%B0%EC%9D%B4%ED%83%80-%ED%83%80%EC%9E%85)를 참고.

```Java
//자바 코드와 비교
int a = 3; //3
double b = 2 * Math.pow(10, 3); //2000.0
float c = 1.6f; //1.6
float inf = (float)Double.POSITIVE_INFINITY; //Infinity
float inf2 = (float)Double.NEGATIVE_INFINITY; //-Infinity
boolean boolt = true; //true 자바에서는 boolean 자료형만이 true, false를 나타낼 수 있다.
boolean boolf = false; //false
Object none = null; //null
```
- 위 코드는 나름대로 자바 코드와 파이썬 코드의 자료형 선언에 대해 비교한 것이다. 그렇다. 자바에 비하면 엄청나게 간결하게 코드를 작성할 수 있다는 장점이 있다. 하지만, 코드가 길어지면 변수의 타입이 헷갈리거나 하는 이유로 찾기 힘든 에러를 마주할 수도 있다.. 적절한 주석과 함께 적절한 변수명을 사용해서 작성되어야 하는 것을 알 수 있다. 아래 코드는 전혀 다른 타입인 리스트로 선언된 a 변수에 정수형 1을 할당한 코드이다. 하지만, 전혀 에러 없이 잘 작동한다.

```python
a = [1, 2, 3]
a = 1
```

### 연산자
위에서 자료형에 대해 알아보았으니, 다음은 파이썬의 연산을 위한 연산자에 대해서 알아보자.

연산자|설명|예
:-:|:-:|:-:
+, -, &#42;, /|더하기, 빼기, 곱하기, 나누기| a = (1 + 2 - 3 &#42; 4) / 5, 결과: -2
&#42;&#42;, //, %|제곱, 소숫점은 버리고 몫만 취함, 나머지| a = 2 &#42;&#42; 3 // 3 % 4, 결과: 2
==, !=, >, <, <=, >=|등호, 같지 않음, 부등호|if a == 1:<br> print("a는 1")
&#42;=, +=, -=, /=, %=, //= |피연산자와 연산 결과를 할당한다.| a = 2 <br> a &#42;= 2 <br> print(a), 결과: 4
and, or, not|모두 참이면 참, 둘 중 하나만 참이여도 참, 참,거짓 반전| ret = a == 1 and "a is 1" or "a is not 1" <br> print(ret), 결과: a is 1
&, &vert;, ^, ~, <<, >>|비트 단위 연산. 각각 AND, OR, XOR, Complement, Shift|a = 10<br>b = 20<br>c = a &amp; b<br>print(c), 결과: 0
in, not in|피연산자가 컬렉션에 있는지 확인| a = [1,2,3]<br>b = 1 in a<br>print(b), 결과: True
is, is not|같은 Object를 가리키는지 확인| a = 1<br>b = a<br>print(a is b), 결과: True

연산자는 타언어에서 지원하는 연산자 종류와 크게 다를바가 없다. 하지만, &#42;&#42; 라던지 // 와 같은 추가된 연산자와 ++, --와 같이 지원하지 않는 연산자는 주의해야 할 점이다. 문자열에서도 &#42;는 굉장히 유용하게 사용된다. 그렇다면, 자료형 중에서 가장 자주 다뤄지는 문자열 자료형에 대해서 알아보자.

### 문자열 자료형
우선 문자열은 말그대로 문자열을 저장하기 위해 사용되며 따옴표 '문자열' 혹은 큰따옴표 "문자열를 사용해서 선언하게 되며, 문자열을 개행해서 여러 줄로 나열하고 싶을땐 타언어에서 처럼 \n를 사용한 개행을 하거나 아래 코드처럼 사용할 수도 있다.
```python
s = '''문자열
'''
```

일반적으로 어떤 언어에서나 문자열을 가공해서 사용하기 위해서 포멧팅이라는 방식을 사용하게 되는데, 파이썬에서도 역시 문자열 포멧팅을 자주 사용하게 된다. 포멧팅이란, 포멧을 미리 정해놓고 나중에 그 값을 대입하는 형식으로 작성하는 것을 말한다.

문자열 포멧팅 사용법은 다양하게 있지만, 가장 기본이 되는 것은 예약된 키워드인 &#37; 기호를 문자열에 넣고, &#37; 기호를 한 번 더 쓴 상태에서 원하는 문자열을 순서에 맞게 대입하는 형식이다. &#37; 앞뒤로 각각 하나의 값만을 받아들이므로 만약 &#37; 뒤의 값이 복수 개이면 튜플(소괄호)로 묶어주어야 한다.
```python
# 파이썬으로 작성한 문자열 포멧팅 코드
a = 'Life is %s %s' % ('too', 'short')
print(a) # Life is too short
```
```java
// 자바로 작성한 문자열 포멧팅 코드
String a = String.format("Life is %s %s", "too", "short");
System.out.println(a); //Life is too short
```
문자열뿐만 아니라, 정수나 소수도 포멧팅 대상이 될 수 있다. 지시어의 내용은 아래 표를 참고하자.


지시어|의미
:-:|:-:
%s|문자열 (String)
%c|문자 1개(character)
%d|정수 (Integer)
%f|부동소수 (floating-point)
%o|8진수
%x|16진수
%%|Literal % (문자 % 자체)


또한, 문자열은 +나, &#42; 기호를 통해 여러 문자열을 하나로 만들거나, 동일한 문자열을 중복 생성할 수도 있다.

```python
string1 = "문자열"
string2 = "문자열"
print(string1+string2) # 문자열문자열
print(string1*2) #문자열문자열
```

문자열과 관련된 함수는 굉장히 다양한데 Java 언어에서 표준 라이브러리로 지원하지 않는 함수도 상당수 존재한다. 문자열과 관련된 함수에 자세한 내용은 [문자열 포스팅](https://seongjaemoon.github.io/2017/12/05/pythonStringFormat/)을 참고.

### 조건문과 반복문
#### 조건문 (if, elif, else, pass)
조건문은 특정 조건이 주어졌을 때 이를 판단하고 상황에 맞게 분기하기 위한 방법으로 사용된다. 파이썬에서는 switch~case 구문이 주어지지 않기 때문에, 오직 if~elif~else 구문으로 해결해야 한다. 조건문은 특정 상황에 맞게 분기해야 하므로, 조건을 거는 순서가 굉장히 중요하다. 또한 파이썬에서는 pass라는 예약 어를 지원하는데, 이는 특정 조건문을 만났을 때 pass를 통해 아무것도 실행하지 않고 분기할 수 있게 하는 예약 어이다.
```python
if 조건 :
	# 조건에 맞는 실행문
elif 다른 조건 :
	# 위 조건과 다른 조건 실행
elif 다른 조건 :
	pass # 아무것도 실행하지 않고 조건문 종료
else :
	# 위 조건에 모두 해당하지 않으면 실행
```
- 참고로 파이썬에는 타 언어에서 볼 수 없는 표현의 종류가 있는데, Java에서의 3항 연산자와 비교하면 한눈에 구분된다. if 구문의 경우 a라는 변수가 결과 값임에도 if 구문 앞에 나오는 독특한 형태의 문법으로 작성된다.

```python
a = 7
b = a == 7 and 'good' or 'not good'  # good
b = 'good' if a == 7 else 'not good' # good
```
```java
int a = 7;
String b = a == 7 ? "good" : "not good"; //good
```
#### 반복문 (while, for, break, continue)
반복문은 이름 그대로 특정 상황을 반복적으로 연출하기 위해서 사용되며, 조건문과 더불어 코드에 대부분을 차지하는 구문이다. 전체적인 코드의 실행 시간과 직결되는 부분이 많으며, 필요에 따라 다양한 연출이 가능하다. while문과 for문은 특정 조건을 만족할 때 구문을 계속해서 반복하게 할 때 사용된다. 이때, break문과 continue 문은 조건문과 함께 사용되어 각각 반복문을 바로 종료하거나, 코드를 더 실행하지 않고 다시 반복문 처음으로 돌아가게 하는 예약 어이다.
```python
# 파이썬으로 작성한 1~10까지 홀수합 구하기 while문 코드
sum = 0
i = 0
while i < 10:
    i += 1
    if i % 2 == 0:
        continue
    sum += i

print(sum) # 25
```
```java
//자바로 작성한 1~10까지 홀수합 구하기 while문 코드
int i = 0;
int sum = 0;
while(i < 10) {
	i += 1;
	if(i % 2 == 0)
	continue;
	sum += i;
}
System.out.println(sum); //25
```
for문은 while문과 마찬가지로 반복적인 작업을 진행하기 위해 사용되지만, 다음 포스팅에서 알아볼 컬렉션 자료형과 함께 유용하게 사용할 수 있다. 우선 기본적으로 for문을 사용할 때 유용한 함수인 range 함수에 대해 알아보자.

range() 함수는 입력받은 조건에 맞는 숫자 객체를 만들어 값을 반환한다. 기본적으로 초기값, 최댓값, 증감 값으로 구성되며 초기값이 주어지지 않을 경우 인덱스는 0부터 시작한다. 주의할 점은, range의 반복 요소는 range 함수에 선언한 최대값의 n-1까지만 반환한다는 점이다. 아래 표와 같이 사용법은 총 세 가지이다.


예|의미|반환값
:-:|:-:|:-:
range(3)|최댓값|0, 1, 2
range(3,6)|초기값, 최댓값|3, 4, 5
range(2,11,2)|초기값, 최댓값, 증감값|2, 4, 6, 8, 10


입력값의 길이를 계산해서 반환하는 len()함수도 for문과 자주 사용되므로, 함께 기억해두자. 아래 코드는 for문을 이용한 간단한 코드이다.
```python
luv = "iluvu" # 길이 5
for i in range(len(luv)):
	print("I luv u") # iluvu 5번 출력
```
```java
String luv = "iluvu"; //길이 5
for(int i = 0; i < luv.length; ++i){
	System.out.println("I luv u") //iluvu 5번 출력
}
```


### 기본 코드 테스트
마지막으로 파이썬으로 작성된 버블정렬과 자바로 작성된 버블정렬 코드를 비교해보자. 상대적으로 파이썬 코드가 짧고 간결하게 느껴진다.
```python
# 파이썬으로 작성한 버블정렬 코드
import random

a = random.sample(range(1, 10), 5) # 랜덤한 리스트 컬렉션 만들기, 컬렉션과 관련된 더 자세한 내용은 다음 포스팅에서!

# 파이썬은 메인 함수가 존재하지 않아도 바로 스크립트를 실행할 수 있다.
print(a)

def bubbleSort(a):
    l = len(a)
    for i in range(1, l):
        for j in range(l-1):
            if a[j] > a[j+1]:
                a[j], a[j+1] = a[j+1], a[j]


bubbleSort(a) # 함수와 관련된 자세한 내용은 다음 포스팅에서!

print(a)
"""
[1, 8, 6, 5, 3]
[1, 3, 5, 6, 8]
"""
```
```java
//자바로 작성한 버블정렬 코드
package com.javaTest;

public class Test {

	public static void main(String[] args) {
		int[]a = new int[5];

		for(int i = 0; i < a.length; ++i) {
			a[i] = new Random().nextInt(10)+1;
		}
		System.out.println(Arrays.toString(a));
		bubbleSort(a);
		System.out.println(Arrays.toString(a));

	}

	public static void bubbleSort(int[] a) {
		int len = a.length;
		for(int i = 1; i < len; ++i) {
			for(int j = 0; j < len - 1; ++j) {
				if(a[j] > a[j+1]) {
					int temp = a[j];
					a[j] = a[j+1];
					a[j+1] = temp;
				}
			}
		}
	}

	/*
	 [9, 9, 2, 6, 7]
	 [2, 6, 7, 9, 9]
	*/

}
```

그렇다면 파이썬은 과연 어느 곳에 적용해서 쓰일 수 있을까?
1. Web & Internet Development
2. Educational Advancment
3. Scientific Studies/Computing
4. Desktop development
5. Numeric Computing
6. Software development
7. Business Application development
8. Machine Learning
9. IOT
10. Game Development
11. Rapid Prototyping
12. Browser Automation
13. Data analysis
14. Image Processing

그렇다.. 거의 웬만한 곳에서는 모두 쓰일 수 있다. 수학적 연산의 강력함 때문에 머신 러닝과 이미지 프로세싱 작업 등에 많이 활용되고, 장고 프레임워크를 이용해 웹 개발 또한 가능한 강력한 언어인 것이다! 이렇게 간단하게? 파이썬 기본 자료형과 기초 문법에 대해 정리해보았다. 기초 문법은 우선은 이 정도만 제대로 정리되면 필요한 것은 반복된 연습일 것으로 생각된다.

다음은 파이썬의 컬렉션 자료형과 함수, 패키지 관리 시스템인 pip와 웹 크롤러 개발을 위한 파이썬의 표준라이브러리, 서드파티 라이브러리 등에 대해 알아보는 걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)

참고 사이트<br>
* [The Hitchhiker's Guide to python](http://docs.python-guide.org/en/latest/)
* [freeCodeCamp guide python](https://guide.freecodecamp.org/python/data-structures)
* [The Python tutorial](https://docs.python.org/3.6/tutorial/index.html)
* [예제로 배우는 Python 프로그래밍](http://pythonstudy.xyz/)
