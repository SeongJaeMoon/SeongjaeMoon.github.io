---
layout: post
title:  "파이썬 패키지 및 라이브러리 정리"
date:   2018-04-06 23:00:00
author: Seongjae Moon
categories: Python
tags:   Python 파이썬 패키지 pip 라이브러리  Atom Java
---

[저번 포스팅](https://seongjaemoon.github.io/python/2018/04/06/python-course3.html)에 이어서 파이썬 기초부터 웹 크롤러 개발 네 번째 포스팅을 진행해보자. 이번 포스팅에선 파이썬의 패키지와 크롤러 개발 및 데이터 분석을 위한 라이브러리 설치에 대한 내용을 간단하게 알아보자. 테스트 코딩용 파이썬 에디터는 [Atom](https://atom.io/)과 script 패키지를 이용해서 작성하였고, [Python 3.6.4](https://www.python.org/downloads/) 버전을 사용했다. Java 코드 테스트는 마찬가지로 [Atom](https://atom.io/)과 JDK 8 환경에서 코딩되었다.

# 파이썬 패키지 구조
파이썬에서 모듈은 하나의 .py 파일을 가리키며, 패키지는 이러한 모듈들을 모은 컬렉션을 가리킨다. 파이썬의 패키지는 하나의 디렉토리에 놓여진 모듈들의 집합을 가리키는데, 그 디렉토리에는 일반적으로 &#95;&#95;init&#95;&#95;.py 라는 패키지 초기화 파일이 존재한다. (Python 3.3 이후부터는 init 파일이 없어도 패키지로 인식이 가능하다)

패키지는 모듈들의 컨테이너로서 패키지 안에는 또다른 서브 패키지를 포함할 수도 있다. 파일시스템으로 비유하면 패키지는 일반적으로 폴더에 해당하고, 모듈은 폴더 안의 파일에 해당한다. 특정 패키지를 구성하고, 이러한 패키지를 사용하기 위해서 import 구문을 확장해서 사용한다.

아래 코드는 test라는 패키지의 서브 패키지인 util 패키지에서 math.py라는 모듈이 존재하고 math 모듈 안에 adder 함수를 사용하기 위한 두 가지 방법을 보여준다.
```python
from test.util import math # 모듈안의 모든 함수 import.
math.adder(1, 1) # from 전체 패키지명 import 모듈명.

from test.util.math import adder # 특정 함수만 import.
adder(1, 50) # from 전체 패키지명.모듈명 import 함수명.
```
 특정 폴더 안에 있는 &#95;&#95;init&#95;&#95;.py 파일은 해당 폴더가 일반 resource 폴더가 아닌 패키지 폴더임을 명시하는 역할을 할 수 있다. 패키지 3.3+ 이하 버전과의 호환을 위해 사용하는 것이 좋다. &#95;&#95;init&#95;&#95;.py 파일 안에는 &#95;&#95;all&#95;&#95; 이라는 리스트 변수가 존재하는데, 이 변수에 사용할 모듈 파일 명을 할당하면 해당 모듈의 사용을 명시한다. 아래 코드는 &#95;&#95;all&#95;&#95; 변수에 특정 math 모듈을 할당한 예이다.
```python
# __ini__.py 의 내용.
__all__ = ['math']
```
# 파이썬 패키지 관리자 pip
우선 pip는 파이썬으로 작성된 패키지 소프트웨어를 설치 · 관리하는 패키지 관리 시스템 이다. 파이썬 2.7.9 이후 버전과 파이썬 3.4 이후 버전은 pip를 기본적으로 포함한다. pip의 장점은 CLI 환경에서 간편하게 파이썬 패키지를 설치하고 삭제할 수 있다는 점이다. 윈도우에서는 명령 프롬프트를, 맥 OS에선 터미널을 통해 pip를 설치할 수 있다.
```
sudo pip3 install --upgrade pip
```
위 명령어는 맥 OS에서 pip를 업그레이드 하기 위한 명령어이다. pip 버전은 지속적으로 업그레이드 되므로 특정 패키지를 다운로드 하던 도중 "You should consider upgrading via the 'pip install --upgrade pip' command."와 같은 구문을 마주하게 되면 위 명령어로 업그레이드 해주는게 좋다. 또한, 윈도우에서는 관리자 버전으로 프롬프트를 실행하고, 맥에선 sudo 명령어를 통해 설치를 진행한다.

# 크롤러 개발을 위한 라이브러리

## requests
```
pip3 install requests
```
특정 웹 페이지에 HTTP 요청을 보낼 수 있도록 사용되는 모듈이다. GET, POST 방식 모두 지원하며 헤더에 특정 값을 실어 함께 보낼 수 있다.
## bs4
```
pip3 install bs4
```
requests 모듈을 통해 불러온 웹 페이지 정보를 HTML 문법으로 분석이 가능하게 해주는 모듈이다. HTML 엘리먼트, 속성에 대한 접근을 CSS 선택자 형식 및 함수를 통한 컨텐츠에 접근이 가능하다.
## Numpy matplotlib Scipy Sympy (선택)
```
pip3 install numpy
pip3 install matplotlib
pip3 install Scipy
pip3 install sympy
```
수학 및 과학 연산을 위한 파이썬 패키지이다. 여러가지 구현하기 어려운 함수들을 보다 간편하게 구현할 수 있게 도와주며, 데이터 저장 형식 및 그래프 등 시각화 구현도 도와준다.<br><br>

간단하게 파이썬 패키지와 스크래핑과 데이터 분석을 위한 라이브러리에 대해 정리해보았다. 역시 파이썬은 강력한 3rd party 라이브러리를 많이 지원하고 있는 것을 알 수 있다. 다음 포스팅에선 본격적으로 특정 웹 페이지의 원하는 데이터를 크롤링하기 위한 HTML 태그 문법과 스크래핑 테스트 코드를 작성해보는 걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)

- 참고 사이트
- [pip 위키 백과](https://ko.wikipedia.org/wiki/Pip_(%ED%8C%A8%ED%82%A4%EC%A7%80_%EA%B4%80%EB%A6%AC%EC%9E%90))
- [The Hitchhiker's Guide to python](http://docs.python-guide.org/en/latest/)
- [freeCodeCamp guide python](https://guide.freecodecamp.org/python/data-structures)
- [The Python tutorial](https://docs.python.org/3.6/tutorial/index.html)
- [예제로 배우는 Python 프로그래밍](http://pythonstudy.xyz/)
