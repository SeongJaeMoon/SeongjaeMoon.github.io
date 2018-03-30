---
layout: post
title:  "Git 기본 명령어 정리2"
date:   2017-12-09 23:00:00
author: Seongjae Moon
categories: Git
tags:   Git Github 명령어
---

Github를 통해 자신만의 repository를 구성하고 소스코드의 백업용 저장소로 운용할 수 있지만, 협업 툴로써 여럿이 공동 작업을 하고 이를 합치거나 이전 상태로 돌리는(롤벡) 작업을 편리하게 할 수 있다. 사실, Docker나 Jenkins와의 콜라보와 더불어 후자의 경우가 버전 관리의 이점으로 Git 매니아를 양산하는 이유일 듯 싶다.

하지만, Git에도 큰 단점이 있으니.. 처음에 배우기가 너무 어렵다는 것이다. 내가 이 만큼의 시간을 투자하면서 이 짓?을 해야되나 하는 막막함이 들기 쉽상이다. ~~나도..~~ 그래서 협업의 기능으로 어떤 것들이 가능한지 차근차근 정리하고자 한다.

### 1. Git 기본 사용 루틴

[전 포스팅](https://seongjaemoon.github.io/git/2017/12/01/gitPrinciple.html)에서 알아봤듯이, 우리가 저장소를 새로 만들고 저장소에 저장하는 과정은 크게 4가지 단계를 거치는 것을 알 수 있었다.

##### 저장소 생성 > add (스테이징 영역)> commit (원격 저장소에 올릴 준비)> push (원격 저장소에 올리기)
내 로컬 영역에 작업을 진행하고 커밋까지 진행한 후, 푸쉬를 통해 실제 원격 저장소에 저장한다는 단순한?루틴이다. 여기서 스테이징 영역이란, 작업디렉터리에 변경부분만 따로 임시로 저장하는 공간이라 생각하면 되겠다.

* Centralized Workflow
* Feature Branch Workflow
* Gitflow Workflow
* Forking Workflow

열심히 구글링 해 본 결과, 크게 위 네 가지의 Git을 활용한 협업 워크 플로우가 존재하는 것을 알 수 있었다.  (자세한 내용은 [여기](https://www.atlassian.com/git/tutorials/comparing-workflows)를 참고) 여러가지 협업 방법이 존재 하겠지만, 나름대로 제일 간단한 방법?을 살펴보고자 한다.

## Branch
우선 Git의 중요 컨셉 중 하나인 브랜치에 대해 정리해보자. 쉽게 예로 들자면, A는 사용자 UI용 클래스를 만들고, B는 데이터베이스와 관련된 작업을 맡고 있다고 하자. 각자가 맡은 기능 개발에 대해 Branch를 구성해서 작업을 진행할 수 있다. 우선 관례적으로 공식적인 변경 이력을 관리하기 위해서 **master** (프로젝트 관리자) 브랜치를 사용한다. **master 브랜치의 내용이 최종 내용이라고 생각하면 되겠다.**

그리고 위 처럼 **기능별 브랜치를 따로 구성하여 ui-service나 dao-v01 등의 브랜치 이름**을 사용하여 개발자별 작업을 구분짓고, 기능 개발 완료 시점에 master 브랜치에 merge(병합)요청을 진행하면 된다. 브랜치 관련 bash 명령어는 [여기](https://git-scm.com/book/ko/v1/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EA%B4%80%EB%A6%AC)를 참고하자. 여기서 협업에서 중요한 **Pull request**란 것에 대해 잠깐 알아보고 넘어가자.  

## Pull request
브랜치를 이용하면 격리된 영역에서 안전하게 새 기능을 개발할 수 있을 뿐만 아니라, 풀 리퀘스트를 이용해서 브랜치에 대한 팀 간 코드 리뷰를 이끌어 낼 수도 있다.(실제로 Github에선 열띤 토론의 장이 열린다는..) 간단하게 말해, 기능 개발을 끝내고 master에 바로 병합하는 것이 아니라, 브랜치를 중앙 저장소에 올리고 **master에 병합해달라고 요청하는 것**이 풀 리퀘스트다. 사실 여기까지 봐도 잘 모르겠다. 백문이 불여일견이라고, 직접 해보는게 가장 빠르다.

## Workflow 구성해보기.
먼저, 시나리오를 구성해보자. 성재와 그의 친구들은 이번에 새롭게 도서 관리 프로그램을 만드는 프로젝트를 시작했다.  중앙 저장소 master 브랜치에 최종 프로젝트 내용을 저장하고, 각자 맡은 역할에 대해 기능 개발이 완료되면 이를 중앙 저장소에 합치는 식으로 진행하기로 했다.

### 1. 개발 하기.
성재는 관리자 역할을 하기 때문에, 전체적으로 설계만 하고 실제 코딩은 그의 친구들이 주로 맡고 있다.  ~~량아치~~~ 우선 그의 친구 중 한명의 Github에 중앙 저장소로 사용할 Repogitory를 생성했다고 가정하자.

#### 1-1.  Fork하기
우선, 중앙 저장소의 내용을 Fork해서 개인 프로젝트 저장소에 가져와야 한다. ~~니꺼 내꺼, 내꺼 니꺼 시전~~
![저장소 Fork](/assets/uploads/gitFork1.jpg)
내 Repository에 Fork된 저장소를 확인할 수 있다.
![저장소 확인](/assets/uploads/gitForkMe.jpg)
Fork 된 저장소는 나의 개인 원격 저장소와 똑같이 작업할 수 있다. (이 원격 저장소 변경 내용이 Fork한 기존 저장소의 내용에 영향을 주지 않는다는 뜻!)
#### 1-2. Clone 및 개발하기
작업을 하기 위해 bookmng를 ssh를 통해 clone해서 내가 원하는 로컬 저장소로 불러 온다. 주소는 각자 저장소 URL에 맞는 주소를 복붙해주면 된다.
```bash
$ git clone git@github.com:SeongJaeMoon/bookmng.git
```
![로컬 저장소 확인](/assets/uploads/gitLocalMoon.jpg)
로컬 저장소에 저장된 파일의 변경 사항 추가 및 코딩을 열심히 작업한다.
#### 1-3 Push하기
기능 개발을 완료했다면, 이제 나의 원격 저장소에 올려주면 끝!
```bash
$ git add . //스테이징 영역에 올리기
$ git commit -a -m 'setting bookmng' //setting bookmng란 메세지로 커밋하기
$ git push origin master // 원격 저장소의 master 브랜치에 푸시하기
```
### 2. Pull Request 하기
![원격 저장소 확인](/assets/uploads/gitForkandPush.jpg)
내가 맡은 부분에 개발이 완료되면, 이 부분에 대해 중앙 저장소에 Merge(소스 합치기) 해달라고 요청해야 한다. 이 작업을 웹 상에서 가능하게 하기 위해 Github에서 Pull Request를 제공하고 있다. 위 사진의 New pull request를 클릭하면, 아래와 같은 화면이 나타나게 된다.

![Pull Request1](/assets/uploads/gitPR1.jpg)
Comparing changes 화면이 나타나면 Create pull request를 클릭하여 새로운 Request를 만든다.

![Pull Request2](/assets/uploads/gitPR2.jpg)
어떤 브랜치의 변경 내용을 Pull Request 할지 선택할 수 있으며(여기선 편의상master branch 사용), 내가 변경한 내용에 대한 내용을 쭈우욱 적어준다. 그리고 나서 Create pull request를 누르면 끝!

![Pull Request3](/assets/uploads/gitPRfinish.jpg)
그러고 나면 Pull Request가 생성된다. File changed에서 어떤 변경사항이 있는지 웹 상에서 확인 가능하고, 코멘트를 달거나 하는 등의 팀원 간 코드 리뷰가 가능하다. 최종적으로 중앙 원격 저장소 관리자가 Pull Request를 허락해주면 저장소에 변경 내용이 적용된다!!  

### 3. 변경사항 로컬에 적용하기
이제 중앙 원격 저장소에 저장된 내용과 내 로컬 저장소에 내용을 합쳐 최종 변경본을 가지고 추가 작업을 해야한다.  
```bash
$ git pull --rebase origin master //git pull 명령으로 중앙 저장소의 변경 이력 로컬 저장소로 내려 받기
```
위의 rebase 옵션 없이 쓸 수도 있지만, 불필요한 병합 커밋을 한 번 더해야 하는 번거로움이 있으므로, rebase 옵션을 쓰는 것이 좋다. merge와 rebase에 대한 더 자세한 내용은 [여기](https://backlog.com/git-tutorial/kr/stepup/stepup1_4.html)를 참고.

여기서 살펴본 내용은 지극히 작업이 정상적으로 이루어졌을 경우를 대상으로 하고 있다. 실제 협업을 하다보면 수 십~수 백 개의 커밋이 오갈 수 있고, 커밋을 이전 상태로 돌리거나 병합 충돌 등의 경우를 해결해야 할 경우가 분명.. 발생할 것이다. 이는 구글링으로 해결 해보는걸로~

[다음](https://seongjaemoon.github.io/git/2018/01/08/gitPrinciple3.html)은 Github 워크 플로우에 대해  마무리 해보는걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* [Git Fork에서 Pull Request까지](https://medium.com/axisj/github-fork-%EC%97%90%EC%84%9C-pull-request-%EA%B9%8C%EC%A7%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-merge-a22bdd097283)
* [Git의 모든것](http://goodtogreate.tistory.com/entry/Git%EC%9D%98-%EB%AA%A8%EB%93%A0%EA%B2%831-%EA%B8%B0%EC%B4%88-%EA%B0%9C%EB%85%90?category=440231)
