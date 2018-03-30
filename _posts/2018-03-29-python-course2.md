---
layout: post
title:  "파이썬 컬렉션 자료형 및 함수 정리"
date:   2018-03-28 15:00:00
author: Seongjae Moon
categories: Python
tags:   Python 파이썬 자료형 문법 Atom Java
---

저번 포스팅에 이어서 파이썬 기초부터 웹 크롤러 개발 포스팅을 진행해보자. 이번 포스팅에선 리스트 컬렉션과 함수에 대해서 알아보자. 테스트 코딩용 파이썬 에디터는 [Atom](https://atom.io/)과 script 패키지를 이용해서 작성하였고, [Python 3.6.4](https://www.python.org/downloads/) 버전을 사용했다. Java 코드 테스트는 마찬가지로 [Atom](https://atom.io/)과 JDK 8 환경에서 코딩되었다.

# 컬렉션 자료형
우선 컬렉션 자료형에 대해서 알아보자. 파이썬에선 타언에서처럼 배열이라는 참조 타입이 존재하지 않는다. 대신 굉장히 강력한? 컬렉션 데이터 타입으로 모든 것을 해결할 수 있다. 기본적으로 자바나 C++ 등에서 활용되는 컬렉션 자료형과 유사하다고 보면 된다. 그렇다면, Java 코드와 비교를 통해 공통점과 차이점, 사용법 등에 대해 정리해보자.

## 리스트(List) 자료형
파이썬의 리스트는 동적배열로서 프로그래머가 인덱스를 조절하지 않아도 값을 자유롭게 추가 삭제할 수 있는 구조를 갖는다. 리스트 안의 요소(Element)들은 그 값을 자유롭게 변경할 수 있는 가변(Mutable) 데이터 타입이다. 자바 언어에선 다형성을 위해 List 컬렉션에 주로 ArrayList를 구현하는 방식으로 코딩한다면, 파이썬에선 리스트 컬렉션만으로 관련된 일련의 작업을 모두 해결 가능하다. 리스트는 대괄호[]로 감싸 선언하며, 다양한 타입의 데이터를 하나의 리스트가 가질 수 있다. 아래 코드를 통해 리스트의 기본 선언 방법과 사용법에 대해 정리해보자.

```python
# 빈 리스트 선언
a = []
# 리스트에 값 할당
a = [1, '2', 3.0, False]
for i in a:
  print(i, end=' ') # 1 2 3.0 False
```
```java
import java.util.*;

public class Sample001{

    public static void main(String[] agrs){
      //자바 리스트 Object형 구현
      List<Object> a = new ArrayList<Object>();

      a.add(1);
      a.add("2");
      a.add(3.0);
      a.add(false);

      Object obj = new Object();
      //캐스팅과 박싱을 통한 형변환 필요.
      for(int i = 0; i < a.size(); ++i){
        obj = a.get(i);
        if(obj instanceof String){
            obj = (String)a.get(i);
            System.out.print(obj+" ");
        }else if(obj instanceof Integer){
            obj = (Integer)a.get(i);
            System.out.print(obj+" ");
        }else if(obj instanceof Double){
            obj = (Double)a.get(i);
            System.out.print(obj+" ");
        }else if(obj instanceof Boolean){
            obj = (Boolean)a.get(i);
            System.out.print(obj+" ");
        }
      }
    }
    //1 2 3.0 false
}
```

위 코드는 파이썬과 자바에서 다양한 타입을 갖는 리스트에 값을 할당하고, 출력하는 예를 보여준다. 일반적으로 리스트는 같은 타입을 갖는 값을 여러 개 묶어서 사용하기 위해 선언하므로, 예시로만 알아두면 될 것 같다. 여기서 눈여겨 볼 것은 in 이라는 키워드이다.

- 파이썬에선 멤버쉽 연산자라 불리우는 in 이라는 키워드가 존재한다.
> if 구문에서 사용되면 컬렉션에 해당 값이 존재하는지 확인하여 True, False를 반환하고, 반복문에서 사용하면 컬렉션에 값을 하나하나 읽어올 수 있다.

기본 선언 방법에 대해 알아보았으니, 실제 값의 추가, 삭제 및 값의 접근 등과 관련된 리스트 컬렉션의 사용법에 대해 계속해서 정리해보자.

### 리스트 인덱싱(Indexing)
먼저, 자주 쓰이는 리스트의 인덱싱에 대해 알아보자. 리스트 인덱싱이란, 특정 요소에 값을 인덱스로 접근하기 위해 사용하는 방법으로 타 언어에서 배열에서 "변수명[인덱스]"처럼 인덱스로 접근하는 것을 생각하면 바로 이해가 가능하다. 마찬가지로 인덱스는 0부터 시작한다.
```python
# 리스트 인덱싱 테스트
a = [1, 2, 3, 4]
print(a[0]) # 1

for i in range(len(a)):
    print(a[i], end='') # 1234
```
### 리스트 슬라이싱(Slicing)
리스트 슬라이싱이란, 리스트에 들어있는 값들 중 특정 부분만 따로 떼서 선택하는 방법이다. 주로 다수의 값 중 조건에 만족하는 값을 검색하고 수정, 삭제 등을 하기 위해 사용한다. 인덱스는 0부터 시작하며, range 함수처럼 마지막 요소의 n-1 까지만 슬라이싱 한다. 또한, 처음 인덱스가 생략되면, 0부터 시작되며, 마지막 인덱스가 생략되면, 리스트의 끝까지 포함됨을 의미한다. 표현법은 변수명[시작 인덱스:끝 인덱스+1]처럼 표현한다.
```python
a = [1, 2, 3, 4]

print(a[0:]) # 1, 2, 3, 4 (0~끝)
print(a[:1]) # 1 (0~(1-1))
print(a[1:3]) # 2, 3 (1~(3-1))
print(a[:-1]) # 1, 2, 3 (0~(1-(-1)))
```
### 리스트에 값 추가, 삭제, 갱신
리스트에 값을 추가하거나 삭제, 갱신하기 위해서 각각 정해진 키워드를 사용한다. 값을 삭제하기 위해서 del 변수명[인덱스]처럼 선언하고, 추가는 append 메서드를 이용해 추가한다. 자바에 add 메서드와 같이 리스트에 마지막에 추가되며, 갱신은 리스트에 인덱스를 직접 지정하며 특정 값을 할당하여 갱신한다.
```python
a = [1, 2, 3, 4]

del a[3] # 삭제
print(a) # 1, 2, 3
a.append(4) # 추가
print(a) # 1, 2, 3, 4
a[3] = 5 # 갱신
print(a) # 1, 2, 3, 5
b = a # b 변수에 a 리스트 할당
'''
전 포스팅에서 살펴본 문자열 타입과 마찬가지로
+, * 연산자로 다른 리스트를 병합하거나
하나의 리스트의 값을 중복해서 확장할 수 있다.
'''
print(a+b) # [1, 2, 3, 5, 1, 2, 3, 5]
print(a*2) # [1, 2, 3, 5, 1, 2, 3, 5]
```
### 리스트 관련 메서드
리스트 관련 함수를 정리하기 전에, 함수와 메서드를 따로 구분 짓는 이유가 궁금해진다. ~~아님 어쩔 수 없다..~~ 그렇다면 그 기준에 대해서 알아보자.
> 메서드 : 객체에 속해있는 함수로 정의된다. 때문에 위에 append 메서드는 리스트 컬렉션 객체에 속해있다고 할 수 있다.


> 함수 : range, print 등과 같이 특정 객체에 속해 있지 않은 것은 함수로 정의된다. 일반적으로 사용자가 def 키워드를 이용해 정의하는 사용자 정의 '함수'를 예로 들 수 있다.

리스트 메서드는 clear, pop 등과 같이 다양한 메서드가 존재한다. 여기선, 검색을 위해 사용되는 index와 count에 대해서만 알아보자. 리스트 안에 특정 요소의 인덱스를 확인하기 위해서 index() 메서드를 사용하고, 특정 요소가 몇 개 있는지 체크하기 위해서 count() 메서드를 사용할 수 있다.
```python
a = "Life is too short".split()
print(a) # [Life, is, too, short]
print(a.index("Life")) # 0
print(a.count("Life")) # 1
```
위에 사용된 spilt() 메서드는 문자열 객체에서 사용되는 메서드로서 구분자를 통해 값을 구분하여 이를 리스트로 반환하는 함수이다. 더 자세한 내용은 [여기]("https://seongjaemoon.github.io/python/2017/12/05/pythonStringFormat.html")를 참고하자.

마지막으로, 리스트를 생성하는 방법에는 여러 가지 방법이 있겠지만, 파이썬에서는 독특한 방법으로 원하는 형태의 리스트를 생성할 수 있다. 우선, 표현 방법은 "변수명 = [표현식 for 요소 in 컬렉션 [if 조건식]]"처럼 선언 가능하며, 아래 예는 1~10까지 수 중에서 짝수만을 갖는 리스트를 생성하는 코드이다. 더 자세한 내용은 아래 컬렉션 내포에서 살펴보자.
```python
a = [i for i in range(1, 11) if i % 2 == 0]
print(a) # [2, 4, 6, 8, 10]
```
파이썬에선 많은 양의 기본 내장 함수를 지원한다. 함수 관련 부분은 그때 그때 필요할 때 정리해보자. 아울러, def 키워드와 관련된 내용은 밑에서 더 알아보고, 계속해서 튜플 자료형에 대해서 알아보자.

## 튜플(Tuple) 자료형
튜플은 리스트와 비슷하게 여러 요소들을 갖는 컬렉션이다. 리스트와 다른 점은 튜플은 새로운 요소를 추가하거나 갱신, 삭제하는 일을 할 수 없다. 즉, 튜플은 한변 결정된 요소를 변경할 수 없는 불변(Immutable) 데이터 타입이다. 선언 방법은 소괄호()를 이용해서 선언한다. 또한, 튜플의 요소가 하나밖에 없을 경우엔 콤마,를 붙여 명시적으로 튜플임을 표시해야 한다. 또한, 튜플의 값은 인덱스에 맞춰 서로 다른 변수에 동시에 할당 가능하다. 할당 방법은 아래 코드를 통해 알아보자.
```python
a = (1, 2, 3)
print(a) # 1, 2, 3
a[0] = 1 # 불가
del a[0] # 불가
a =(1) # 일반 int 타입
a =(1,) # 튜플 컬렉션
a = ("seongjae", "moon")
s, m = a
print(s,",", m) # seongjae , moon
```
### 튜플 인덱싱, 튜플 슬라이싱, 튜플 병합
튜플은 리스트와 마찬가지 방법으로 인덱싱, 슬라이싱, +, * 연산자를 통한 병합 및 중복 생성 등이 가능하다. 위에서 내용을 정리했으니, 예제 코드로만 살펴보자.
```python
a = (1, 2, 3)
print(a[0:]) # 1, 2, 3
print(a[-1]) # 3(-는 뒤에서부터 요소를 찾는다.)
print(a + a) # 1, 2, 3, 1, 2, 3
print(a*2) # 1, 2, 3, 1, 2, 3
```
## 셋(Set) 자료형
셋은 중복이 없는 요소들 (unique elements)로만 구성된 집합 컬렉션이다. 자바의 Set 컬렉션을 생각하면 쉽게 이해가 가능하다. 중괄호{}를 이용해 선언하며, 중복된 값을 할당하거나 초기값으로 설정해도, 중복된 값이 저장되지 않는다. 이러한 특성 때문에, 중복 없이 유니크한 값들의 집합만을 원할 경우 유용하게 사용할 수 있다.
```python
a = {1, 1, 2, 2, 3, 3}
print(a) # 1, 2, 3
```
### 셋 메서드
셋은 리스트나 튜플처럼 순서가 있는 컬렉션 타입이 아니기 때문에, 인덱스를 통한 값의 접근이나 수정 및 슬라이싱 등은 불가하다. 때문에 순서성을 갖는 셋 컬렉션을 만들기 위해선 따로 사용자 정의 함수로 만들어야 한다. 하지만, 마찬가지로 값의 추가, 삭제는 가능하다.
```python
a = {1, 2, 3}
a.add(4)
print(a) # 1, 2, 3, 4
a.remove(4)
print(a) # 1, 2, 3
a.update({1, 2, 3, 4, 5}) # 한 번에 여러 요소 추가
print(a) # 1, 2, 3, 4, 5
a.clear() # 값 모두 삭제
```
### 셋 집합 연산
셋 자료형은 수학에서 사용하는 서로 다른 집합 간의 연산이 가능하다. & 연산자를 이용한 교집합, | 연산자를 이용한 합집합, - 연산자를 이용한 차집합을 들 수 있다. 각각의 연산자는 union, intersection, diffrence 메서드로 대체 가능하다.
```python
a = {1, 2, 3}
b = {1, 2, 4}
print(a&b) # 1, 2
print(a|b) # 1, 2, 3, 4
print(a-b) # 3
```
## 딕셔너리(Dictionary) 자료형
딕셔너리는 키(Key)와 값(Value)을 하나의 요소로 갖는 컬렉션이다. 딕셔너리는 흔히 Map 이라고도 불리는데, 키(Key)로 신속하게 값(Value)을 찾아내는 해시 테이블(Hash Table) 구조를 갖는다. 자바에서 Map 컬렉션을 생각하면 쉽게 이해가 가능하다. 선언 방법은 셋과 마찬가지로 중괄호{}를 사용하며, {키:값} 형태로 콜론:을 통해 키와 값을 구분 짓는다.
```python
a = {"name":seongjae, "age":26}
print(a["name"]) # seongjae
print(a["age"]) # 26
```

### 딕셔너리 값 접근, 추가, 삭제, 갱신
딕셔너리는 변수명[키]을 통해 값에 접근하고, 해당 키값에 값을 할당하거나 새로운 키 값에 값을 할당하여 값을 추가할 수 있다. 리스트와 마찬가지로 del 키워드를 통해 해당 키에 해당하는 키, 값을 삭제할 수 있다. 또한 앞서 살펴본 반복문과 in 연산자를 통해 반환 받은 키를 통해 값에 접근할 수 있다.
```python
py = {"Life": 1, "is": 2, "too": 3}
py["seong"] = 4 # 수정
py["short"] = 5 # 추가
del py["Life"] # 삭제
print(py) # 'is': 2, 'too': 3, 'seong': 4, 'short': 5

a = {"a": 1, "b": 2, "c": 3}
for key in a:
  print(a[key], end = '') # 123
```
### 딕셔너리 관련 메서드
딕셔너리 클레스엔 다양한 메서드들이 존재하는데 유용한 메서드들이 존재한다. 먼저, keys()는 딕셔너리의 키값들로 된 dict_keys 객체를 리턴하고, values()는 딕셔너리의 값들로 구성된 dict_values 객체를 리턴한다.
```python
a = {"a": 1, "b": 2, "c": 3}
# keys
keys = a.keys()
for k in keys:
    print(k, end='') #abc

# values
values = a.values()
for v in values:
    print(v, end='') #123
 ```
딕셔너리에 키를 통해 값을 접근할 때, 변수명[키] 형식으로 접근할 경우 값이 없을 때는 에러가 발생할 수 있다. 하지만, 딕셔너리 클래스의 get 메서드를 이용하면 에러가 아닌 None 타입을 반환받을 수 있다.
```python
a = {"a": 1, "b": 2, "c": 3}
print(a["d"]) # KeyError: 'd' 에러 발생
print(a.get("d")) # None
```
## 컬렉션 생성자, 컬렉션 내포, 이터레이터
각각의 컬렉션은 update, clear, copy 등과 같이 서로 다른 컬렉션 타입이 공통적인 기능을 갖는 메서드들이 존재하므로 하나만 잘 숙지해두면 두고두고 여러 컬렉션 타입에서 사용 가능하다. 또한, 각각의 컬렉션은 생성자를 통해 서로 다른 컬렉션으로 타입 변환이 가능하다. 생성자 선언 방법은 각각의 클래스 이름을 통해 선언 가능하다. 예를 들어 list(), dict(), set(), tuple()와 같은 식이다.
```python
a = [("a", 1), ("b", 2), ("c", 3)]
b = dict(a) # 튜플을 요소로 갖는 리스트 -> 키:값 형태의 딕셔너리 컬렉션으로 변환
print(b) # {'a': 1, 'b': 2, 'c': 3}
a = b.items() # dict 클래스의 items 메서드를 통해 dict_items 형으로 변환
print(list(a)) # [('a', 1), ('b', 2), ('c', 3)] 리스트 생성자를 통해 리스트로 변환
print(set(a)) # {('b', 2), ('c', 3), ('a', 1)} 셋 생성자를 통해 셋 타입으로 변환
print(tuple(a)) # (('a', 1), ('b', 2), ('c', 3)) 튜플 생성자를 통해 튜플 타입으로 변환
```
파이썬의 컬렉션 타입은 컬렉션 내포(Comprehension)란 것이 존재한다. 파이썬 2 버전에선 리스트 컬렉션만 지원하고, 파이썬 3 버전부턴 딕셔너리와 셋 컬렉션도 지원한다. 쉽게 말해, 특정 조건(지정된 표현식)에 맞게 새로운 컬렉션을 빌드하는 것을 말한다. 앞서 리스트 컬렉션에 대한 설명 중 "변수명 = [표현식 for 요소 in 컬렉션 [if 조건식]]"을 통해 새로운 리스트를 만들었었다. 이를 컬렉션 내포라고 할 수 있다. 딕셔너리 타입과 셋 타입의 Comprehension를 코드를 통해 알아보자.
```python
# 리스트 -> 셋으로 변환시, 특정 조건을 통해 새로운 셋 생성 가능.
testList = [1, 2, 2, 3, 3, 4, 4]
testSet = {i*i for i in testList} # 셋을 만들 때 중괄호{} 감싼다.
print(testSet) # {4*4, 1*1, 2*2, 3*3} -> {16, 1, 4, 9}

# 딕셔너리의 키와 값을 서로 바꾼 형태의 새로운 딕셔너리를 출력하는 예
testDict = {"a":1, "b":2, "c":3}
newDict = {val:key for key, val in testDict.items()} # 딕셔너리를 만들 때 중괄호{}로 감싸고, 키:값 형태로 표현식을 작성한다.
print(newDict) # {1: 'a', 2: 'b', 3: 'c'}
```
위와 같이 Python Comprehension 표현 방법을 이용하면 특정 조건에 맞는 새로운 컬렉션을 만들어낼 수 있어 유용하게 사용할 수 있다. 파이썬에서도 자바나 C++ 등과 같이 Interator 클래스가 존재한다. ~~객체 지향 만세!~~ 파이썬에서는 또한 Generator라는 것이 존재하는데, 계속해서 이터레이터와 제너레이터에 대해서 알아보자.

먼저, 리스트, 셋, 딕셔너리와 같은 컬렉션이나 문자열과 같은 문자 Sequence 등은 for 문을 써서 하나씩 데이터를 처리할 수 있는데, 이렇게 하나하나 처리할 수 있는 컬렉션이나 입력 Sequence 들을 Iterable 객체(Iterable Object)라 부른다. Iterable 객체는 사용자가 직접 '클래스'를 구성해서 만들 수 도 있다. 여기선, 목표 범위를 넘어서니 파이썬 오브젝트와 관련된 내용을 정리할 때 다루기로 하고, 컬렉션 타입에서 이터레이터를 어떻게 사용할 수 있는지 살펴보자.

파이썬에 기본적으로 내장된 iter 함수를 이용해 이터레이터를 구현할 수 있으며, iter("Iterable객체") 식으로 표현한다. 아래 자바 코드와 비교를 통해 사용법을 확인해보자.
```python
# 파이썬 코드로 작성된 iter 함수를 이용한 리스트 접근
a = [1, 2, 3, 4, 5]
it = iter(a)
# next 함수를 이용해서 객체의 값에 순서대로 접근할 수 있다.
print(next(it)) # 1
print(next(it)) # 2
# for 문 사용가능.
for i in it:
    print(i, end='') # 345
```
```java
import java.util.*;

public class Sample001{

    public static void main(String[] agrs){

      List<Integer> a = new ArrayList<Integer>();
      a.add(1);
      a.add(2);
      a.add(3);
      a.add(4);
      a.add(5);
      //자바 코드로 작성된 Iterator 클래스 사용법.
      Iterator<Integer> it = a.iterator();
      System.out.println(it.next()); //1
      System.out.println(it.next()); //2
      while(it.hasNext()){
        System.out.print(it.next()); //345
      }
    }
}
```
파이썬에는 Generator라는 키워드가 존재한다. 제너레이터는 Iterator의 특수한 한 형태이다. Generator 함수(Generator function)는 함수 안에 yield 키워드를 사용하여 데이터를 하나씩 반환하는 함수이다. 제너레이터 함수가 처음 호출되면, 그 함수 실행 중 처음으로 만나는 yield 에서 값을 리턴한다. Generator 함수가 다시 호출되면, 직전에 실행되었던 yield 문 다음부터 다음 yield 문을 만날 때까지 문장들을 실행하게 된다. 이러한 Generator 함수를 변수에 할당하면 그 변수는 generator 클래스 객체가 된다.
```python
def gen(): # gen 함수 정의 및 yield 키워드를 통해 Generator 함수로 선언.
    yield 1
    yield 2
    yield 3
    yield 4
    yield 5

g = gen() # generator 객체를 변수에 할당 가능
print(next(g)) # 1
for i in g:
    print(i, end='') # 2345

```
# 함수
함수(Function)는 특정 코드블록에서 실행되는 코드 구문으로, 전체 적인 코드의 흐름을 제어하고, 중복되는 코드를 제거하기 위해 사용할 수 있다. 파이썬에서는 함수를 정의하기 위해 def 라는 키워드를 사용하며, 타언어와 마찬가지로 인자로 넘어오는 파라미터 값과 반환(return) 값은 존재할 수도 있고 없을 수도 있다.

## 기본 형식
위에서 살펴본 대로 파이썬에서 함수는 def 키워드를 정의한다. 함수명은 사용자가 직접 정의하며, 의미있는 이름을 사용하는 것이 좋다. 기본 형식은 아래와 같이 사용하면 된다.
```python
def sum(a, b): # def 함수명(파라미터)
  return a + b # 반환값

print(sum(3, 5)) # 8
```
## Default parameter
파이썬에서 기본 파라미터라는 것이 존재한다. 함수에 파라미터는 정의되어 있지만, 파라미터가 주어지지 않을 경우 기본 값으로 설정하는 파라미터라고 할 수 있다. 사용 방법은 "파라미터 명 = 기본값" 형식으로 선언한다.
```python
def defaultSum(a, b, default = 1):
  return a + b + default

print(sum(1, 2)) # 4
```
## Variable-length parameter
파이썬에서 함수의 파라미터를 넘겨받을 때, 파라미터의 갯 수가 정해져 있지 않을 수 있다. 그럴 땐 자바에서 "변수명..." 를 이용해 가변 파라미터를 사용하듯이, 파이썬에선 에스터 리스크* 기호를 이용해 가변 길이 파라미터를 선언할 수 있다.
```python
def vlSum(fl, *number):
  sum = 0
  for i in range(len(number)):
    sum += i
  return sum + fl

print(vlSum(1.0, 1, 2, 3, 4, 5)) # 11.0
```
## Named parameter
파이썬에서는 이름 있는 파라미터라는 것이 존재 한다. 사용법은 함수에서 사용한 "인자명 = 값" 형식으로 함수에 파라미터를 넘겨줄 수 있다. 가독성 있고 직관적인 코드를 작성할 수 있는 장점이 있다. 실제 코드를 보면 바로 이해할 수 있다.
```python
def namedSum(one = 1, two = 2, three = 3):
  return one + two + three

print(namedSum(one = 1, two = 2, three = 3)) # 6
```
## Return type
파이썬에서는 하나의 함수에 여러 반환 값이 존재할 수 있다. 실제로는 하나의 튜플을 반환하는 것이지만, 파이썬 튜플의 특성을 이용해 하나의 함수에서 동작한 여러 작업에 대한 반환 값을 여러개 가지게 할 때 사용하면 유용하다.
```python
def returnSum(*number):
  sum = 0
  for i in range(len(number)):
    sum += i
  return sum, len(number)

print(returnSum(1, 2, 3, 4, 5)) # (10, 5)
```

이상으로 간단하게? 파이썬의 컬렉션과 함수에 대해 정리했다. 내용이 부족한 면이 있지만, 파이썬이라는 프로그래밍 언어의 다양한 여러 내용을 압축해서 정리한 것이므로 더 자세한 내용은 실제 코드를 작성하면서 응용해보는 걸로~ 다음은 파이썬 패키지 관리자와 라이브러리에 대해 알아보는 걸로!

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)

- 참고 사이트
- [The Hitchhiker's Guide to python](http://docs.python-guide.org/en/latest/)
- [freeCodeCamp guide python](https://guide.freecodecamp.org/python/data-structures)
- [The Python tutorial](https://docs.python.org/3.6/tutorial/index.html)
- [예제로 배우는 Python 프로그래밍](http://pythonstudy.xyz/)
