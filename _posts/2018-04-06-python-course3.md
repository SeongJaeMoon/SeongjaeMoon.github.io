---
layout: post
title:  "파이썬 클래스 및 모듈 정리"
date:   2018-04-05 15:00:00
author: Seongjae Moon
categories: Python
tags:   Python 파이썬 클래스 모듈 문법 Atom Java
---

[저번 포스팅](https://seongjaemoon.github.io/python/2018/03/28/python-course2.html)에 이어서 파이썬 기초부터 웹 크롤러 개발 세 번째 포스팅을 진행해보자. 사실 파이썬에선 객체지향적인 성격보단 그 자체로 바로 실행되는 스크립트 성질만으로 충분히 원하는 결과를 만들어낼 수 있다. 그래도 알아두면 좋을 테니 정리해보는 걸로~ 그렇다면 거두절미하고, 이번 포스팅에선 파이썬의 클래스와 모듈에 대해 자바 코드와 비교하며 간단하게 알아보자. 테스트 코딩용 파이썬 에디터는 [Atom](https://atom.io/)과 script 패키지를 이용해서 작성하였고, [Python 3.6.4](https://www.python.org/downloads/) 버전을 사용했다. Java 코드 테스트는 마찬가지로 [Atom](https://atom.io/)과 JDK 8 환경에서 코딩되었다.

# 파이썬 클래스
파이썬에서는 기본적으로 OOP(Object-Oriented Programming)를 지원한다. 파이썬에서 사용되는 모든 것들은 객체로 이루어져 있으며, 객체로 처리된다. 하지만, 파이썬은 인터프리터 언어로 일반적인 main 함수(또는 메서드)가 없어도 실행이 되며, 동적 바인딩을 통한 객체에 값을 할당한다. 때문에 C 언어와 같이 클래스가 없이 함수들의 집합으로만 이루어지게 코드를 작성할 수 있지만, 알아두면 좋을 객체 지향의 장점을 살릴 수 있는 클래스와 객체에 대해 간단하게 알아보자.

## 클래스의 선언
파이썬에서 클래스를 선언하기 위해선 "def 클래스명():"를 이용해 클래스를 선언하며, 이름 명명 규칙은 일반적으로 카멜 케이스를 따른다. 아래 예는 MyCar라는 간단한 클래스를 선언하는 코드이다.

```python
def MyCar():
  pass
```
위 코드에서는 클래스 안에 작성된 코드가 없는 빈 클래스이므로 pass 키워드를 통해 아무 일도 하지 않는 것을 명시한다.

## 클래스 구조
어떤 언어든 각 언어마다 고유한 특징이 존재하듯 파이썬도 다른 언어엔 없는 특별한 키워드들이 많이 존재하는데, 그 중 클래스를 구성하는데 있어서 어떤 차이가 있는지 자바 코드와 비교하며 하나씩 알아보자.

```python
class MyCar():
    engine = 0 # 클래스 변수

    def __init__(self, speed, tire): # 생성자
        # self.* # 인스턴스 변수
        self.speed = speed
        self.tire = tire
        MyCar.engine += 1


    @staticmethod
    def is_engine(speed1, speed2):
            return speed1 == speed2


    @classmethod
    def get_engine(cls):
      return cls.engine


my_car = MyCar(5, '한국')  

print(my_car.speed) # 5
print(my_car.tire) # 한국
print(my_car.get_engine()) # 1
```
```java
class MyCar{
  private static int engine = 0;
  private int speed;
  private String tire;

  public MyCar(int speed, String tire){
    this.speed = speed;
    this.tire = tire;
    MyCar.engine += 1;
  }

  public void setSpeed(int speed){
      this.speed = speed;
  }
  public int getSpeed(){
    return speed;
  }
  public void setTire(String tire){
    this.tire = tire;
  }

  public String getTire(){
    return tire;
  }

  public static int getEngine(){
    return MyCar.engine;
  }

  public static void main(String []args){
      MyCar myCar = new MyCar(5, "한국");

      System.out.println(myCar.getSpeed()); // 5
      System.out.println(myCar.getTire()); // 한국      
      System.out.println(myCar.getEngine()); // 1
  }
}
```
위 코드는 파이썬과 자바에서 MyCar라는 클래스를 선언할 때의 예를 유사하게 표현한 것이다. 우선 self 키워드에 대해 알아보자. self 키워드는 자바의 this 키워드와 유사하며, 객체 자신의 특정 변수를 인스턴스 메서드의 인자로 받는 값으로 할당하기를 원할 때 사용한다. 인스턴스 메서드는 자기 자신을 가리키는 self 키워드를 인자의 첫 번째에 항상 가지고 있어야 한다.

다음으로, 파이썬은 접근 제한자가 없다. 그렇기 때문에 기본적으로 모두 public 상태이며, my_car 라는 인스턴스의 멤버에 닷(.) 연산자로 바로 접근이 가능하다.

아울러, 관례적으로 언더스코어(&#95;)를 붙인 변수는 private 상태로 취급하고(원하면 직접 접근 가능하다.), 언더스코어를 두 개(&#95;&#95;)를 붙인 변수나 메서드의 경우에만 private 상태로 명시하여 접근 불가하게 할 수 있다.

마지막으로 &#95;&#95;init&#95;&#95; 키워드는 생성자(Constructor)를 명시하는 전용 키워드로, 객체가 생성될 때 자동으로 호출되는 메서드를 의미한다. 파이썬 메서드명으로 &#95;&#95;init&#95;&#95; 을 사용하면 이 메서드는 생성자가 된다. 객체의 초기화를 위해 사용되는 것은 타언어와 마찬가지이지만, 약간 다른 성질을 갖는다. 더 자세한 내용은 파이썬 런타임 엔진에 대해 구글링? 해보는 걸로~

### staticmethod, classmethod
다음은 위 코드에서 &#64; 데코레이터(자바의 어노테이션과 같은 역할)가 붙은 두 가지 메서드에 대해서 알아보자. 먼저, 정적 메서드는 &#64;staticmethod를 메서드명 앞에 작성하여 정적 메서드임을 명시하고, self 파라미터를 갖지 않고 인스턴스 변수에 액세스 할 수 없다. 따라서, 정적 메서드는 보통 객체 필드와 독립적이지만 로직상 클래스 내에 포함되는 메서드에 사용된다. 클래스 메서드는 메서드 앞에 @classmethod라는 데코레이터를 표시하여 해당 메서드가 클래스 메서드임을 표시한다.

다음으로 클래스 메서드는 정적 메서드와 비슷한데, 객체 인스턴스를 의미하는 self 대신 cls라는 클래스를 의미하는 파라미터를 전달받는다. 위에 예와 같이 클래스 메서드는 cls를 인자로 받아 클래스 변수에 접근할 수 있다. 일반적으로 인스턴스 데이터에 접근할 필요가 없는 경우 클래스 메서드나 정적 메서드를 사용하는데, 이때 보통 클래스 변수를 액세스 할 필요가 있을 때는 클래스 메서드를, 이를 액세스 할 필요가 없을 때는 정적 메서드를 사용한다.

### &#95;&#95;add&#95;&#95;, &#95;&#95;cmp&#95;&#95;, &#95;&#95;del&#95;&#95;, &#95;&#95;sub&#95;&#95;, &#95;&#95;str&#95;&#95;
파이썬에는 Initializer(&#95;&#95;init&#95;&#95;) 이외에도 객체가 소멸될 때 (Garbage Collection 될 때) 실행되는 소멸자(&#95;&#95;del&#95;&#95;) 메서드, 두 개의 객체를 ( + 기호로) 더하는 &#95;&#95;add&#95;&#95; 메서드, 두 개의 객체를 ( &#45; 기호로) 빼는 &#95;&#95;sub&#95;&#95; 메서드, 두 개의 객체를 비교하는 &#95;&#95;cmp&#95;&#95; 메서드, 문자열로 객체를 표현할 때 사용하는 &#95;&#95;str&#95;&#95; 메서드 등 많은 특별한 용도의 메서드들이 있다.

아래 코드는 위에서 선언한 MyCar라는 클래스의 speed, tire 인스턴스 변수를 더하기(&#43; 기호로) 할 수 있도록 &#95;&#95;add&#95;&#95; 메서드를 사용한 예이다. (자바에선 연산자 오버로딩이 불가.)
```python
def __add__(self, other):
      obj = MyCar(self.speed + other.speed, self.tire + other.tire)
      return obj

my_car = MyCar(5, '한국')

print(my_car.speed) # 5
print(my_car.tire) # 한국
print(my_car.get_engine()) # 1

my_car2 = MyCar(5, '금호')

my_car3 = my_car + my_car2

print(my_car3.speed) # 10
print(my_car3.tire) # 한국금호
print(my_car3.get_engine()) # 3
```
이 밖의 클래스 상속이나 다형성, 다중 상속 등의 대한 내용은 필요한 부분이 나오면 따로 정리하는 걸로 하고, 계속해서 파이썬 모듈에 대해서 알아보자.

# 파이썬 모듈
모듈(Module)은 파이썬 코드를 논리적으로 묶어서 관리하고 사용할 수 있도록 하는 것으로, 보통 하나의 파이썬 .py 파일이 하나의 모듈이 된다. 자바 언어에서 클래스를 import 해서 사용하는 것과 같은 방법이다. **모듈 안에는 함수, 클래스, 혹은 변수들이 정의될 수 있으며, 실행 코드를 포함할 수도 있다.** 이러한 모듈들을 사용하기 위해서는 모듈을 import구문을 이용하여 사용하면 되는데, import 문은 다음과 같이 하나 혹은 복수의 모듈을 불러들일 수 있다.
## 모듈 import
```python
import 모듈명[,모듈명, 모듈명 ... N]
```
여러 개의 모듈을 import 할 경우 위 코드 처럼 모두 한 라인에 작성할 수 있지만 PEP8을 보면 모듈명은 한 라인에 하나씩 작성하는 것을 권장한다.
## from 모듈 import 함수
하나의 모듈.py에는 여러 개의 사용자 함수 및 내장 함수 등이 존재할 수 있는데, 이 중 하나의 함수만을 불러 사용하기 위해서는 아래와 같이 "from 모듈명 import 함수명"이라는 표현을 사용할 수 있다. 특정 함수명을 명시하지 않고 asterisk(&#42;)를 사용할 수 있는데, 이렇게 하면 해당 모듈에 속한 모든 함수를 불러올 수 있다. 하지만, PEP8에선 권장하지 않는 표현이라고 나와있다.
```python
from time import strftime
```
위의 예는 파이썬에서 자주 사용되는 time이라는 모듈 중에서 원하는 포멧에 맞게 출력하기 위해서 사용되는 strftime이라는 함수를 import한 예이다. 아래 코드의 예를 보면 알 수 있듯이, 모듈명만을 이용하여 import 하면 "모듈명.함수" 형식으로 함수를 호출해야 하지만, 함수를 직접 import 하면 모듈명을 작성하지 않고 곧바로 함수에 접근할 수 있다.
```python
import time
from time import strftime

print(time.strftime("%Y-%m-%d")) # 2018-04-06
print(strftime("%Y-%m-%d")) # 2018-04-06
```
만약 함수의 명칭이 길거나 직관적이지 않을 경우 "from 모듈명 import 함수명 as 약어" 형식으로 "약어.함수명"로 함수를 호출할 수 있다.
```python
from time import strftime as s
print(s("%Y-%m-%d")) # 2018-04-06
```
## 모듈 생성
기존에 파이썬에 정의된 내장 함수나 외장 함수를 import 해서 사용할 수 있지만(사실 대부분을 차지한다.), 불가피하게 원하는 형식으로 함수를 작성해야 하거나 자주 사용되는 함수 로직 등을 util.py 등에 저장해두고 필요할 때 마다 호출해서 사용하면 굉장히 유용하다. 아래 코드는 사용자가 module.py라는 파이썬 코드를 작성하고, module.py에 작성된 사용자 정의 함수인 adder를 호출한 예이다.
```python
from moudel import adder

print(adder(1,2)) # 3
```
파이썬에서 모듈을 import 하면 그 모듈을 찾기 위해 다음과 같은 경로를 순서대로 검색한다.

1.현재 디렉토리
2.환경변수 PYTHONPATH에 지정된 경로
3.Python이 설치된 경로 및 그 밑의 라이브러리 경로

때문에 하나의 패키지에서 자주 사용되는 모듈은 되도록 같은 디렉토리에 작성하고, 여러 패키지에서 사용되는 모듈은 sys.path 파이썬 경로 디렉터리에 추가해주는게 좋다. 패키지에 대한 자세한 내용은 다음 장에서 정리하도록 하고, sys.path를 통해 모듈의 경로를 추가하는 방법에 대해 알아보자.  

sys.path를 사용하기 위해서는 sys라는 시스템 모듈을 import 해야 하며, sys.path는 임의로 수정할 수도 있다. 예를 들어, 기존 sys.path에 새 경로를 추가(append)하면, 추가된 경로도 이후 모듈 검색 경로에 포함된다.
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
>>>sys.path.append("C:\\경로명")
```

## &#95;&#95;main&#95;&#95;
위의 예에서 module.py라는 모듈을 import 하고 adder 사용자 함수를 호출하여 더하기 연산을 진행하였다. 이렇게 import를 통한 단순 함수 호출도 가능하지만, 모듈 단위로 전체 스크립트를 바로 실행할 수도 있다. 파이썬에서 하나의 모듈을 import 하여 사용할 때와 스크립트 전체를 실행할 때를 동시에 지원하기 위하여 흔히 관행적으로 모듈 안에서 &#95;&#95;name&#95;&#95; 을 체크하곤 한다.

파이썬의 &#95;&#95;name&#95;&#95; 변수는 파이썬이 내부적으로 사용하는 특별한 변수명이다. 아래 코드의 예처럼 특정 파이썬 파일이 단독으로 컴파일되어 실행될 때 작동될 코드와 해당 파이썬 파일을 다른 파이썬 파일에서 import 하여 사용할 때를 구분 지어 사용할 수 있다. 단톡 스크립트 실행시에만 작동할 코드를 if 구문 아래에 작성하고, 나머지 내용은 따로 구성하면 import할 때 원치 않는 문제가 발생하지 않는다.
```python
# 여러 함수 로직 등 구성.
if __name__ = "__main__":
  # 단독 스크립트 실행시 작동될 코드 작성.
```



이렇게 파이썬의 클래스와 모듈에 대해서 간단하게 알아보았다. 역시나 파이썬에선 자바나 타언어에선 존재하지 않는 독특한 키워드들이 많이 존재한다는 것을 새삼 느낀다. 모든 값들을 객체로 처리하는 특징 때문인지 몰라도 객체끼리 더하거나 빼는 연산자 오버로딩과 비슷한? 작업들이 가능하다. 오늘 정리한 특정 상태 값과 함수 등을 객체화하고 여러 로직이나 함수, 메서드 등을 모듈화하여 논리적으로 잘 묶을 수 있다면, 코딩이 훨씬 수월해질 것 같다. 다음은 파이썬의 패키지 관리자와 웹 크롤링 개발을 위한 라이브러리에 대해 간단하게 알아보는 걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)

- 참고 사이트
- [The Hitchhiker's Guide to python](http://docs.python-guide.org/en/latest/)
- [freeCodeCamp guide python](https://guide.freecodecamp.org/python/data-structures)
- [The Python tutorial](https://docs.python.org/3.6/tutorial/index.html)
- [예제로 배우는 Python 프로그래밍](http://pythonstudy.xyz/)
