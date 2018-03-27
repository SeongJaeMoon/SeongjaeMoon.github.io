---
layout: post
title:  "GitKraken 써보기"
date:   2018-03-10 15:00:00
author: Seongjae Moon
categories: Git
tags:   Git Github GitKraken 크라켄
---

오랜만에 Git과 관련된 포스팅! Github를 통한 코드 관리와 풀 리퀘스트를 통한 개발자들 간의 코드 리뷰를 진행 하는 기초적인 부분에 대해 [리뷰](https://seongjaemoon.github.io/2017/12/01/gitPrinciple/)를 진행 했었다.

git bash를 이용해서 add > commit > push라는 기본적인 패턴에 대해 다뤄보았는데, 오늘은 SourceTree나 Kraken과 같이 bash창에서 하나하나 명령어를 입력해야 되는 번거러움을 덜어주는 GUI 툴 중에 크라켄이라기 보다는 오징어?에 가까운 크라켄 프로그램에 대해 간단하게 알아보고자 한다.

bash보다 Geek스러움이 떨어지는 단점이 있지만, 이 또한 다른 측면에선 Geek스럽지 않을까..? ~~억지~~  

### 1. 일단 크라켄을 설치 하자.
우선 크라켄을 사용하기 위해서 [여기](https://www.gitkraken.com/)에서 크라켄을 다운로드 받아야 한다. 크라켄은 Windows, Mac, Linux 운영체제에서 모두 지원하는 것으로 보인다. 위 홈페이지에 들어가면 GitKraken, GitKraken Pro, GitKraken Enterprise 세 가지 버전이 있는데 우리는 개인 사용자이니 Free 버전을 다운로드 한다.

Pro 버전에는 Merge Conflict Editor라는 기능을 지원하는데, 이거는 진짜 탐난다...

### 2. 크라켄을 실행하기.
크라켄을 다운로드 받고 Github아이디와 연동해서 회원가입을 하자. GitKraken을 사용할 준비가 끝나면, 크라켄을 실행한다. 아주 귀엽게 생긴 오징어?가 우리를 반긴다.
#### 2-1. Github 저장소와 연결하기.
처음 프로그램을 실행하면 자신의 Github 계정과 연결하는 작업이 필요하다. Authentication 창에서 Connect to Github 버튼을 클릭하여 Github 저장소를 사용할 준비를 한다. Success! 창이 나오면 정상적으로 연결이 이루어진 것이다.

연결을 완료하고 나면, bash에서와 마찬가지로 야매?로 사용하기 위해 Generate SSH key and add to Github 버튼을 클릭하여 SSH 키를 생성하거나 기존의 SSH 키를 사용한다.

![Github 저장소와 Kraken 연결하기](/assets/uploads/Kraken/kraken1.png)
위 작업을 완료하고 Repository Management 창을 열면 위와 같은 화면을 만나게 된다. 각각의 역할은 다음과 같다.
- Open: Local(내 컴퓨터)에 있는 저장소를 연결하여 사용할 수 있다.
- Clone: Github에서 클론하여 바로 저장소를 동기화 한다.
- Init: 새로운 저장소를 만들고 연결한다.

그렇다. bash에서 하던 init, clone, remote add 등과 같은 작업을 GUI 프로그램의 장점인 마우스 까딱?을 통해 손 쉽게 진행할 수 있다! 여기선 java로 작성된 간단한 콘솔 프로그램인 sistmng 라는 프로그램을 연결했다.

#### 2-2. Github 저장소 사용하기.
저장소를 연결했으니, 이제 본격적으로 Git의 장점을 이용한 작업을 진행하면 된다.
![Github 저장소와 연동된 GitKraken 작업 화면](/assets/uploads/Kraken/krakenAll.png)
- 파란색으로 감싸진 화면은 커밋, 병합 이력 등의 전체적인 작업 흐름이 나타나는 부분이다.
- 빨간색으로 감싸진 화면은 실제 프로젝트 파일의 정보를 확인할 수 있는 창이며, 소스 코드의 변동 사항이 있을 경우 이를 자동으로 인식하여 add 할 것인지 묻는 창이 나타난다. 특정 파일은 간편하게 선택할 수 있으므로, 스테이징 영역에 올렸다 뺐다 하기 정말 수월하다.
- 보라색으로 감싸진 화면은 현재 작업 트리 상의 최신 상태를 관리하기 위한 것으로 Checkout, Undo, Redo, Stash, Pull 등 다양한 기능을 수행할 수 있게 해준다.
- 초록색으로 감싸진 부분은 화면에서 보이는 것 처럼 Local 영역과 Remote 영역의 Branch가 나타나게 된다.

사실, 여기까지만 봐도 작업 트리를 육안으로 확인할 수 있다는게 엄청난 장점이 되는 것을 느낄 수 있다. 물론 bash 상에서도 로그를 확인하는게 가능하지만, Kraken으로 보는게 더 예쁘니까..?

특히, 작업 트리에 있는 각 브랜치를 드래그 앤 드랍 해서 특정 브랜치의 위치에 가져다 놓으면 Merge나 Rebase 등을 할 것인지 묻는 창이 나타나는데 Conflict가 발생하지 않으면, 순식간에 병합이 일어나는 것을 확인할 수 있다.

![크라켄 저장소 영역 관리](/assets/uploads/Kraken/krakenBranch1.png)
위 화면에서 보이는 것 처럼 마우스 우클릭으로 로컬 영역과 원격 영역의 관리를 마우스 까딱 몇 번으로 해결 가능하다.

![크라켄 작업 트리 관리](/assets/uploads/Kraken/krakenBranch2.png)
마찬가지로 작업 트리 상의 내용에 대한 관리를 마우스 우클릭을 통해 핸들링 가능하다!

![크라켄 풀 리퀘스트 작성](/assets/uploads/Kraken/krakenPR.png)
풀 리퀘스트를 작성하는 것 역시 크라켄에서 가능하다! 오..!

### 크라켄 마무리.
이렇게 깃크라켄에 대해 간단하게 정리해보았다. 모든 기능을 활용해보지는 못했지만, bash 상의 명령어를 하나하나 입력하며 작업을 하는 것보다는 편한점이 확실히 있는 것으로 보인다. Pro 버전의 Merge Conflict Editor가 어떤 기능을 제공하는지 너무 궁금하다.. 역시 돈이 짱인가.. 이 놈의 병합 충돌!!! 새로운 기능을 알아내면 또 포스팅을 진행하도록 해야겠다. 아무튼, bash와 크라켄을 오가며 소스 버전 관리를 열심히 해보는 걸로!

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* [크라켄 홈페이지](https://www.gitkraken.com/)
* [크라켄 가이드](https://support.gitkraken.com/start-here/guide)
* [크라켄 서포트](https://support.gitkraken.com/integrations/github)