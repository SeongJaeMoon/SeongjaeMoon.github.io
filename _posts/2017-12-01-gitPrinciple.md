---
layout: post
title:  "Git 기본 명령어 정리"
date:   2017-12-01 14:56:48
author: Seongjae Moon
categories: Git
tags:   Git Github 명령어
---

Git은 분산 버전 관리 시스템으로, 리누스 토르발스가 리눅스 커널을 관리하기 위해 개발하였다. Git을 만드는 2주간의 기간이 제일 행복했다는 후문이 있다... ~~레알 Geek스럽다.~~ 이러한 Git을 사용하는 프로젝트를 지원하는 웹 호스팅 서비스 중에 Github ~~이 곳은 코더들의 성지~~와 관련해서 포스팅을 하고자 한다.
 
### 1. 일단 Git 설치를 하자.

아래 주소에 접속해서 OS에 맞는 Git을 설치한다.
[Git 설치](https://git-scm.com/downloads)

### 2. Github에 repository를 만든다.

아래 주소에 접속해서 가입을 한다.
[Git 사이트](https://github.com/)

![새로운 Repsitories 만들기 1](/assets/uploads/GitNewBtn.jpg) 
Repsitories 탭에서 위와 같이 생긴 New버튼을 클릭한다. 
![새로운 Repsitories 만들기 2](/assets/uploads/GitCreateRepo.jpg)
New 버튼을 클릭하면 위와 같은 창이 보이게 된다. Repository name 부분에 저장소 이름을 등록하고, Description 부분과 README 부분은 만들어질 저장소의 설명을 다는 부분이다. 나중에 적어도 되므로 안 적어도 그만이다. Public은 전체 공개 프로젝트이고, Private은 비공개 프로젝트이다. ~~쉽게 말해 무료, 유료~~ 적절하게 작성 후 Create repository를 클릭하면 새로운 repository가 생성된다.

### 3.Github와 local repository 연동하기.

Github 페이지와 내 컴퓨터에 저장된 폴더를 연동하기 전에 Git 시스템에 대해 간단하게 정리를 하고자 한다.

### Git은 원격 저장소와 로컬 저장소 두 종류의 저장소를 제공한다. 
> 원격 저장소(Remote Repository): 파일이 원격 저장소 전용 서버에서 관리되며 여러 사람이 함께 공유하기 위한 저장소이다.

> 로컬 저장소(Local Repository): 내 PC에 파일이 저장되는 개인 전용 저장소이다. 

**이 때, 아예 저장소를 새로 만들거나, 이미 만들어져 있는 원격 저장소를 로컬 저장소로 복사해 올 수 있다.**

git을 효율적으로 사용하기 위해 활용할 수 있는 명령어가 굉장히 많다. 하지만 이해하기 너무 어렵다.. git 전용 GUI툴도 있으나, bash를 사용해서 명렁어를 실행하는 것을 권장한다고 한다.~~일단 Geek스럽지 않을 뿐만 아니라, 멋이 떨어진다.~~ 명령어를 이용해서 실제 저장소를 다루기 전에 개념에 대한 간단한 이해가 필요하니 정리해보자.

### 변경을 기록하는 커밋(Commit)
파일 및 폴더의 추가/변경 사항을 저장소에 기록하려면 '커밋'을 해줘야 한다. 커밋을 실행하면 이전 커밋 상태까지의 변경 이력이 기록된 커밋(혹은 리비전)이 만들어진다. 커밋은 시간순으로 저장되며, 최근 커밋부터 거슬러 올라가면 과거 변경 이력과 내용을 알 수 있다. 각 커밋에는 영문/숫자로 이루어진 **40자리 고유 이름이 붙으며, 저장소에선 이 40자리 이름을 보고 각 커밋을 구분한다.** Git의 '커밋'작업은 '작업 트리'에 있는 변경 내용을 저장소에 바로 기록하는 것이 아니라 그 사이 공간인 **'인덱스'**에 **파일 상태를 기록**하여 **'작업 트리'**중 **원하는 파일**만 저장소에 등록할 수 있게 제공한다.

### 원격 저장소에 푸시하기(Push)
내 PC의 로컬 저장소로 변경된 파일을 업로드 하는 것을 Git에서는 (Push)라고 한다. push를 실행하면, 원격 저장소에 내 변경 이력이 업로드되어, **원격 저장소**와 **로컬 저장소**가 **동일한 상태**가 된다.

### 원격 저장소 복제하기(Clone)
원격 저장소의 파일을 내가 원하는 폴더로 복제할 때, 클론(Clone)이라는 작업을 수행한다. Github 페이지에서 일일이 압축 파일로 다운로드 하는 ~~뻘~~짓을 방지할 수 있다.

### 원격 저장소 풀하기(Pull)
원격 저장소를 공유해 여러 사람이 함께 작업을 하면, 모두가 같은 원격 저장소에 푸시(Push)한다. 다른 사람이 원격 저장소에 올려 놓은(Push) 변경 내용을 **내 로컬 저장소에 적용(Pull)**할 필요가 있다. 한 마디로, 공동 작업 중이거나 다른 PC에서 작업한 내용을 업로드 하고 싶을 때 사용한다.

#### 위에서 개념을 간단히 알아 보았다면, 실제 어떤 식으로 git bash를 사용할 수 있는지 정리가 필요하다. 

##### 1. Git bash 사용 전 선행 사항 및 사용할 로컬 Repository(폴더) 설정
원격저장소와 연결하기 전에 개인 경험상 우선 SSH키를 생성해두는 편이 좋다. 나중에 SSH키를 통해서 야매?로 할 수 있는 일이 많아진다.

우선 git bash를 연다. 아래 명령어를 입력한다. (Git bash 상에선 ctr+v 대신 shift+ins로 붙여넣기 한다.)
```bash
$ ls -al ~/.ssh 
```
![새로운 SSH키 생성1](/assets/uploads/gitBashSSH.jpg) 
위 명령어를 입력하고 위와같은 출력 값이 없다면, 새로 만들어야 한다. (ex. id_rsa.pub)
키 생성을 위해 아래 명령어를 입력한다. email은 본인의 github 주소!
```bash
$ ssh-keygen -t rsa -C "your_email@example.com"
```
아래와 같이 비밀번호를 입력하라는 창이 뜨면 입력해주면 끝~  
```bash
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```
키를 만들었으니 에이전트에 추가해야한다. 자세한 설명은 [여기](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)를 참고
```bash
eval “$(ssh-agent -s)”
ssh-add ~/.ssh/id_rsa
```
아래 명령어를 실행하여 키를 클립보드에 저장한다. ctrl + ins를 활용해서 복사해도 된다.
```bash
$ clip < ~/.ssh//id_rsa.pub
```
위 단계까지 정상적으로 끝났다면, 자신의 github 홈페이지에 접속하여, Settings > SSH and GPG keys > New SSH key를 선택한다.
![새로운 SSH키 생성2](/assets/uploads/githubSetting.jpg)  
![새로운 SSH키 생성3](/assets/uploads/githubSSH.jpg) 
위 bash에서 클립보드에 복사한 내용을 key에 집어 넣는다.
![새로운 SSH키 생성4](/assets/uploads/gitSSHkey.jpg)
##### 2. Git bash 명령어 사용
위에서 SSH키를 정상적으로 만들었다면, https와 ssh를 통해서 github에서 클론하거나 push할 수 있다. ~~두 개의 무기를 획득하였습니다.~~
이제 실제로 git bash를 이용해서 내 저장소에 저장해보도록 한다.
```bash
$ mkdir ~/Moon //로컬(내 컴퓨터)에서 사용할 폴더를 만든다.
$ cd ~/Moon //폴더에 들어간다.
```
```bash
$ git init //깃 명령어를 사용할 수 있는 디렉토리로 만든다.
```
```bash
$ git status //현재 상태를 확인한다
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
   //해당 폴더에 파일이 있을 경우에 아래에 빨간 글씨로 파일명과 확장자가 여러개 등장한다.
   modified:   12hours/app.json
   modified:   12hours/package.json
```
```bash
$ git status //현재 상태를 확인한다.
```
```bash
$ git add 화일명.확장자 //깃 주목 리스트에 특정 파일을 add할 수 있다.
$ git add . //이 명령은 현재 파일의 모든 파일을 추가할 수 있다. (변화된 부분을 알아서 추가해준다.)
```
```bash
$ git commit -m “변경 사항등을 적는다.” //커밋해서 변경상황을 찍는다.
```
```bash
$ git remote add origin https://github.com/your_name/your_repo.git //로컬과 github 원격 저장소를 연결한다.
```
```bash
$ git remote -v //연결상태를 확인한다.(아래 처럼 나타나야 정상 작동!)
	origin  git@github.com:your_name/your_repo.git (fetch)
	origin  git@github.com:your_name/your_repo.git (push)
```
```bash
$ git remote rm origin //기존에 연결된 origin을 삭제할 수 있다.
```
```bash
$ git remote set-url origin https://github.com/your_name/your_repo.git //연결된 url을 변경할 수 있다.
```
```bash
$ git push origin master //깃허브로 푸시한다. (실제 github 원격 저장소에 변경사항이 저장된다.)
```
위 작업을 모두 마치고 github에 접속하면 변경사항이 저장된 것을 확인할 수 있다.
##### 3. Repository 확인
![저장소 확인](/assets/uploads/gitMoon2.jpg)

위 그림에서 다운로드 버튼을 클릭한 후 url만 복사한 후, 내가 원하는 저장소에서 마우스 우클릭을 통해 git bash를 실행시킨다. clone 명령어를 이용하면 끝!
```bash
$ git clone https://github.com/any_name/any_repo.git 
```
특정 브랜치(Branch)의 내용만 clone하고 싶을 땐, 아래 형식으로 특정 브랜치만 클론 가능하다. 브랜치에 대한 더 자세한 설명은 다음 포스팅에서 확인할 수 있다. 
```bash
$ git clone -b 브랜치명 --single-branch your_url
```

위 동작처럼 bash를 활용하지 않고 수작업으로 비교적 간단하게 commit을 진행 할 수도 있다. ~~Geek스러움이 떨어지는 단점이있다.~~
Upload files를 클릭하여, 파일을 직접 commit을 진행하면 된다. 간단한 파일을 업로드할 땐 bash보다 효율적이다.  

![저장소 업로드](/assets/uploads/gitUpload.jpg)
choose your files를 선택하여 직접 파일을 업로드하거나 폴더를 여러개 드래그앤드랍하여 한 번에 업로드 가능하다.
단, 한번에 올리는 폴더 갯 수가 100개의 제한이 있다. 파일을 업로드한 후 Commit changes버튼을 클릭하고 잠시 기다리면 커밋이 완료된다.

![저장소 업로드 마침](/assets/uploads/gitUploadFinally.jpg)
아주 잘 되는 것을 확인할 수 있다. 얄루! :)

이로써, github와 git bash 사용법에 대해 간단하게? 알아보았다. 단순한 작업임에도 명령어가 너무 많다... 저장소 생성 -> add -> commit -> push를 기본으로 하는 것을 알 수 있다. 하지만 역시나 언제나 삼천세계에 빠질 우려가 있기 때문에 필요할 때 마다 스택오버플로우나 github document를 읽어가면서 실행해야 함을 알 수 있다. 

[다음](https://seongjaemoon.github.io/2017/12/09/gitPrinciple2/)은 Git의 가장 큰 매력인 branch, pull, merge 및 기타 기능들에 대해 심도있게 살펴 보는걸로~
  
* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* [누구나 쉽게 이해할 수 있는 Git 입문](https://backlog.com/git-tutorial/kr/intro/intro1_1.html)
* [Read the Guide](https://guides.github.com/activities/hello-world/)
* [SSH키 발급](http://nickjoit.tistory.com/94)
