---
layout: post
title:  "파이썬 문자열 메서드 사용법 정리"
date:   2017-12-05 14:24:09
author: Seongjae Moon
categories: Python
tags:   Python 파이썬 문자열 내장메서드
---

파이썬을 공부한지 1주일 정도, java와 비슷하지만 비슷하지 않은 여러 문법과 메서드 때문에 꽤나 애를 먹었다. 1주일밖에 되지 않았지만 아무리 생각해도 귀도 반 로섬은 천재다. 오늘도 Geek스러움에 감탄하며 문자열 관련 메서드를 정리하고자 한다. (코드 예시는 Pycharm을 통해 코딩 되었으며, python 3.6.3 환경에서 작성되었다.)

#### upper()-대문자로 변경
```python
str = "abc".upper()
print(str)

#출력
ABC
```
#### lower() - 소문자로 변경
```python
str = "ABC".lower()
print(str)

#출력
abc
```
#### swapcase() - 대문자는 소문자로, 소문자는 대문자로 변경
```python
str = "aBc".swapcase()
print(str)

#출력
AbC
```
#### capitalize() - 첫 문자를 대문자로 변경
```python
str = "aBc".capitalize()
print(str)

#출력
Abc
```
#### title() - 각 단어의 첫 글자를 대문자로 변경
```python
str = "abc def".title()
print(str)

#출력
Abc Def
```
#### strip() - 문자열 양쪽 끝을 자른다. 제거할 문자를 인자로 전달 (디폴트는 공백)
```python
str = "  abc  "
print(str)
print(str.strip())

#출력
   abc  
abc
```
#### lstrip() - 문자열 왼쪽을 자름
```python
str = "  abc  "
print(str)
print(str.lstrip())

#출력
   abc  
abc
```
#### rstrip() - 문자열 오른쪽을 자름
```python
str = "  abc  "
print(str)
print(str.rstrip())

#출력
   abc
   abc
```
#### replace() - 문자열 특정 부분을 변경
```python
str = "abc"
print(str.replace('a','b'))

#출력
bbc
```
#### format() - 포멧{}을 만들어 놓고 문자열을 생성
```python
str = "{}b{}"
print(str.format('a', 'c'))

#출력
abc
```
#### join() - 리스트 같은 iterable 인자를 전달하여 문자열로 연결
```python
str = ['a','b','c']
print('#'.join(str))

#출력
a#b#c
```
#### partition() - 전달한 문자로 문자열을 나눔(분리), 결과는 튜플(구분자도 포함) cf.이메일, 전화번호, 고유번호 등 저장할 때 유용할 듯.
```python
str = "seongjae.m@gmail.com"
print(str.partition("@"))

#출력
('seongjae.m', '@', 'gmail.com')
```
#### rpartition() - 뒤에서 부터 전달한 인자로 문자열을 나눔
```python
str = "123-456-789"
print(str.rpartition("-"))

#출력
('123-456', '-', '789')
```
#### split() - 전달한 문자로 문자열을 나눔, 결과는 리스트(구분자 포함 안됨)
```python
str = "abc"
print(str.split("b"))

#출력
['a', 'c']
```
#### rsplit() - 뒤에서 부터 전달한 문자로 문자열을 나눔
```python
str = "abcdef"
print(str.split("c"))

#출력
['ab', 'def']
```
#### splitlines() - 라인 단위로 문자열을 나눔
```python
str = """Life is too short
you need python
"""
print(str.splitlines())

#출력
['Life is too short ', 'you need python']
```

다음은 is가 붙은 메서드들이다. 타언어에서 처럼 참 or 거짓 값을 리턴하는 메서드들이다. is까지만 치면 파이참에서 친절하게 자동완성을 해준다.

#### isalnum() - 알파벳 또는 숫자인가?
#### isalpha() - 알파벳인가?
#### isdecimal() - 숫자(decimal, 10진수)인가?
#### isdigit() - 숫자(digit, 10진수)인가?
#### isidentifier() - 식별자로 사용 가능한가?
#### islower() - 소문자인가?
#### isnumeric() - 숫자인가?
#### isspace() - 공백인가?
#### istitle() - title 형식인가? (단어마다 첫 글자가 대문자인가?)
#### isupper() - 대문자인가?

#### count() - 특정 단어(문자열)의 수를 구함 (없으면 0을 반환)
```python
str = "aabc"
print(str.count('a'))

#출력
2
```
#### len() - 문자열의 글자수를 구한다.
```python
str = "abc"
print(len(str))

#출력
3
```
#### startswith() - 특정 단어로 시작하는지 확인
```python
str = "abc"
print(str.startswith('a'))

#출력
True
```
#### endswith() - 특정 단어로 끝나는지 확인
```python
str = "abc"
print(str.endswith('c'))

#출력
True
```
#### find() - 특정 단어를 찾아 인덱스를 리턴 (없으면 -1을 리턴) cf.java의 indexOf()와 유사, 처음 찾은 값만 리턴
```python
str = "abca"
print(str.find('a'))

#출력
0
```
#### rfind() - 뒤에서부터 특정 단어를 찾아 인덱스를 리턴
```python
str = "abca"
print(str.rfind('a'))

#출력
3
```
#### in, not in을 사용하면특정 단어가 있는지 없는지 확인 가능 (True, False)
```python
str = 'a' in 'abc'
print(str)
str = 'd' in 'abc'
print(str)

#출력
True
False
```
#### index() - find와 동일하지만 없을 때 예외를 발생시킴
```python
str = "abca"
print(str.index('d'))

#출력
ValueError: substring not found
```
#### rindex() - rfind와 동일하지만 없을 때 예외를 발생시킴
```python
str = "abca"
print(str.rindex('d'))

#출력
ValueError: substring not found
```
문자열을 다루는건 사람의 자연어 자체를 처리하는 과정이므로 여러 방면에서 굉장히 중요하다. 파이썬에선 이러한 점을 아주*3 많이 감안하여 여러 문자열 처리 메서드를 ~~겁나~~ 제공하는 것 같다.

자바의 스트링 클래스에서도 이 정도 메서드가 지원했는지 기억이 잘 안난다. String.valueOf()면 충분할 줄 알았는데..위 메서드만 잘 기억해도 웬만한 문자열 처리는 가능할 것으로 보인다.

다음 포스팅은 파이썬 자료형과 문법에 대해 정리 하는걸로~  

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* [파이썬 문자열 다루기 참고](http://withcoding.com/74)
* [파이썬 문자열 다루기 참고](https://wikidocs.net/13)
