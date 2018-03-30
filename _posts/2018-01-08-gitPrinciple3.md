---
layout: post
title:  "Git 기본 명령어 정리3"
date:   2018-01-08 23:00:00
author: Seongjae Moon
categories: Git
tags:   Git Github 명령어
---

[전 포스팅](https://seongjaemoon.github.io/git/2017/12/29/gitPrinciple2.html)에 이어서 Github과 관련해 정리 해보고자 한다. 역시 Git 너란 친구는 정리를 해도해도 끝이 없다..  저번엔  Github를 통한 협업 워크 플로우를 간단하게 알아보았다면, 이번 포스팅에선 포크한 원격 저장소의 중간 내용 변경 사항을 내 원격 저장소와 동기화 하는 작업에 대해 알아보고, 더불어 알아두면 유용할 git bash 명령어에 대해 정리하고자 한다.

### 1. Fork 워크플로우
도서 관리 프로그램 개발 협업을 위해 사용했던 워크 플로우에 대해 간단하게 정리해보면,  

##### 1. 원격 중앙 저장소에서 내 원격 저장소로 포크(Fork) 해온다.
##### 2. 내가 맡은 부분을 열심히 개발하고 내 원격 저장소에 커밋(Commit)한다.
##### 3. 코드 리뷰 작성 및 변경 사항에 대한 풀 리퀘스트(Pull request)를 원격 저장소 관리자에게 요청한다.
##### 4. 관리자가 풀 리퀘스트를 받아주면 변경사항을 병합(Merge)한다.
##### 5. 가져오고 합치기(Pull) 명령어를 통해 변경된 최종본을 내 로컬에 저장한다.

### 2. 예외 발생
위와 같은 루틴이 정상적으로 이루어지기 위해선, 내가 개발하는 부분과 연결된 일련의 코드의 변경 사항이 존재하면 안 된다. 하지만, 내가 포크해온 저장소의 원 관리자나 나와 동일하게 포크를 하고 다른 부분에 대해 개발을 진행 중인 개발자가 있을테니 변경 사항이 계속적으로 생기게 될 것이다.

물론 관리자 입장에선 개발 전용 브랜치(Branch) 을 따로 구성해서 개발용 브랜치(dev 등)에 풀 리퀘스트 하도록 하고  최종본 전용(master)  브랜치엔 관리자가 직접 머지하는 식으로 진행하면 크게 문제될 것이 없을 것이다. ~~사실 그렇지 않다..~~

허나 개발자 입장에선, **계속해서 내가 포크해온 저장소에 내용이 바뀐다면, 내가 코딩하고 있는 부분이 무용지물이 될 수도 있다.**

때문에, 포크해온 원격 저장소에 내용을 내 로컬 저장소에 지속적으로 동기화 하는 작업이 필요하다. 변경 사항이 생길 때 마다 일일이 다시 Clone 하는 일은 상상하기도 싫다..  

앞서 Pull 명령어를 이용해 내 로컬에 변경 사항을 적용 가능하다고 정리한 적이 있다.  이에 대해 추가적인 설명을 덧 붙여보자.

#### 1. Fetch & Merge  
먼저 포크해온 원격 중앙 저장소에서 변경 사항을 가져오기(Fetch)를 실행 해야 한다. 가져오기를 진행 하기 전에, 가져올 원격 저장소를 추가로 연결해줘야 한다.
```bash
$ git remote add upstream git@github.com:youmekko/bookmng.git
```
여기선 upstream이라는 이름으로 연결하였는데, 꼭 upstream이 아닌, 다른 이름으로 연결해줘도 상관 없다.  연결 상태를 확인해보자.
```bash
$ git remote -v
origin  git@github.com:SeongJaeMoon/bookmng.git (fetch)
origin  git@github.com:SeongJaeMoon/bookmng.git (push)
upstream        git@github.com:youmekko/bookmng.git (fetch)
upstream        git@github.com:youmekko/bookmng.git (push)

```
아주 연결이 잘 된 것을 확인할 수 있다. 이제 내 로컬 저장소로 가져와보자.
```bash
$ git fetch upstream
remote: Counting objects: 29, done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 29 (delta 15), reused 24 (delta 12), pack-reused 0
Unpacking objects: 100% (29/29), done.
From github.com:youmekko/bookmng
 * [new branch]      master     -> upstream/master
 * [new branch]      seongjae    -> upstream/seongjae
 * [new branch]      youme      -> upstream/youme
```
위처럼 fetch를 받아오면, 자동으로 새로운 브랜치(upstream)을 만들어내고, 이 친구들을 기존의 원격 브랜치(master, 기타 등등 선택할 수 있다.)에 병합(merge)해서 합쳐버리면 된다.
```bash
$ git merge upstream/master
Updating 9a523a0..96b9194
Fast-forward
 src/group1_2.ucls                   |  145 +++++
 src/sist/group1/Library.java        |   46 +-
 src/sist/group1/LibraryDAO.java     | 1075 ++++++++++++++++++-----------------
 src/sist/group1/LibraryService.java |  722 ++++++++++++++---------
 src/sist/group1/User.java           |    2 +-
 src/sist/group1/group1.ucls         |   10 +
 6 files changed, 1161 insertions(+), 839 deletions(-)
 create mode 100644 src/group1_2.ucls
 create mode 100644 src/sist/group1/group1.ucls
```
나의 master 브랜치에 병합이 아주 잘 된 것을 확인 할 수 있다. 여기서 Fast-foward란 나의 변경 사항이 merge하는 것도 커밋 기록에 남게 되는데, 이러한 자질구레한 커밋기록이 남지 않는다. 이를 위해, **중앙 원격 저장소에 마스터 브랜치의 상태와 나의 저장소의 master 브랜치를 항상 동일**하게 유지하면 된다. ~~말은 쉽다.~~

#### 2. Pull
사실 Pull 명령어 만으로도 내 로컬에 변경 이력을 손 쉽게? 적용 가능하다. Pull 명령어는 Fetch 작업과 Merge 작업을 바로 진행하는 것이라고 할 수 있다.  ~~1타 2피 헿~~
```bash
clear@DESKTOP-11M13SN MINGW64 /d/git/bookmng (master)
$ git pull --rebase origin master
remote: Counting objects: 28, done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 28 (delta 19), reused 24 (delta 15), pack-reused 0
Unpacking objects: 100% (28/28), done.
From github.com:SeongJaeMoon/bookmng
 * branch            master     -> FETCH_HEAD
   78bc24a..9a523a0  master     -> origin/master
Updating 78bc24a..9a523a0
Fast-forward
 src/sist/group1/Book.java           |   9 +-
 src/sist/group1/CheckOut.java       |   6 +-
 src/sist/group1/Library.java        |   2 +
 src/sist/group1/LibraryDAO.java     | 283 +++++++++++++++++++-----------------
 src/sist/group1/LibraryService.java | 201 +++++++++++++++++--------
 src/sist/group1/User.java           |   9 +-
 6 files changed, 300 insertions(+), 210 deletions(-)
```
#### 3.Push
이제 우리가 항상 하던  add -> commit -> push를 진행 하면 된다. **'나의 원격 저장소'**에도 마찬가지로, **중앙 원격 저장소와 동기화**가 될 수 있다.
```bash
clear@DESKTOP-11M13SN MINGW64 /d/git/bookmng (master)
$ git add .

clear@DESKTOP-11M13SN MINGW64 /d/git/bookmng (master)
$ git commit -m 'final bookmng'
On branch master
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)
nothing to commit, working tree clean

clear@DESKTOP-11M13SN MINGW64 /d/git/bookmng (master)
$ git push origin master
Counting objects: 19, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (17/17), done.
Writing objects: 100% (19/19), 8.95 KiB | 0 bytes/s, done.
Total 19 (delta 10), reused 0 (delta 0)
remote: Resolving deltas: 100% (10/10), completed with 6 local objects.
To github.com:SeongJaeMoon/bookmng.git
   231efcf..96b9194  master -> master
```

이후 작업은 [전 포스팅](https://seongjaemoon.github.io/2017/12/29/gitPrinciple2/)에서 본 것 처럼, 열심히 개발하고 풀 리퀘스트 하는 일이다 ㅎㅎ

### 알아두면 좋을 git 명령어

#### 1. reflog
가장 최근에 수행한 작업부터 순서대로 작업 히스토리를 본다.
```bash
$  git reflog
```
#### 2. reset
히스토리의 최상단 커밋을 삭제한다.
```bash
$ git reset HEAD@{1}

$ git reset --hard HEAD^^^ //^의 갯 수 만큼 HEAD를 기준으로 삭제할 Commit수 지정
```
#### 3. revert
중간에 삭제할 커밋이 있을 경우 사용하며, 새로운 커밋을 만들어 기존 커밋을 덮어버리기 한다.
```
$ git revert {커밋 ID}
```
#### 4. stash
기존의 마지막 커밋으로 돌아간다. Working Directory는 깨끗해지지만, 내용이 사라지지 않고 담겨있다. (이거 기능 진짜 좋다.)
```
$ git stash
$ git stash list //리스트를 본다
$ git stash pop //리스트에 마지막 작업본을 가져온다. (stash list에서 사라진다.)
$ git stash apply stash@{커밋 ID} //리스트에 특정 작업본을 가져온다. (stash list에서 사라지지 않는다.)
```
Git에 대해 또 간략하게 정리를 해봤다.  Git은 공부를 해도해도 끝이 없다, 하지만 그 만큼 다양한 기능을 제공하는 것일테고 나름 뿌듯함..?도 느낄 수 있다.

기초 개념에 대해 알아 봤으니, 이제 커밋 이력 관리라던지 병합 충돌 회피 등은 단순히 많이 해보는 방법밖엔 없을 것 같다.

역시 가장 중요한건 Git한테 겁 먹지 않는 걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* [Github에서 fork한 저장소 최신 원본으로 동기화 하기](http://lifove.tistory.com/54)
* [Github를 이용하는 전체 흐름 이해하기](https://blog.outsider.ne.kr/866)
* [git의 요술 책갈피, Stash 기능 소개](http://wit.nts-corp.com/2014/03/25/1153)
