---
layout: posts
title:  "Github Action 을 이용하여 Github 블로그 배포하기"
date:   2020-02-29 22:00:00 +0900
comments: true
categories: note
tags: 
  - github
---

* TOC
{:toc}

## 시작하기 전에  

Github Action을 이용하기 전에 나는 블로그를 이런식으로 운영하고 있었다.  
- 블로그의 빌드방식을 github 의존 방식이 아닌 로컬 빌드방식을 이용
- 블로그를 배포하는 방법으로 루비 Rakefile 스크립트 이용  
- 블로그 개발환경을 도커로 구성
> 
[배포 자동화](https://jehuipark.github.io/blog/blog-publish)  
[블로그 개발환경 도커로 구성하기](https://jehuipark.github.io/blog/blog-env-setting-with-docker)

블로그 개발환경을 도커 구성으로 변경하면서, 기존 배포방식에 문제가 발생 하면서 문제점이 보이기 시작했다.  

배포를 Rakefile 태스크에 의존하다 보니, 배포 또한 도커를 이용하게 되었는데 컨테이너 환경에서는 github 에 `push` 하기 위한 권한이 없기 때문에 아래와 같은 추가적인 작업이 필요한 상황이 발생한 것이다.  

- basic credentials 으로 직접 로그인 - **귀찮음**
- 컨테이너에 ssh 설정 - **배포환경이 달라질 때 마다 설정을 해줘야 함**

결정적으로 나는 **개발환경 구축이 되어 있지 않더라도, 배포가 가능한 환경을 원했다.**  
이런 환경이라면, 단순한 수정은 인터넷만 가능하다면 퀵 하게 처리가 가능할 테니까

## Github Action 이 딱이네  
내 요구사항을 처리하기에 딱 좋은 녀석은 얼마전에 회사에서 공유 받은적이 있는 `Github Action` 이 딱이라고 생각하였다.  
나도 `Github Action` 을 사용해본 경험은 없기 때문에, 사전조사를 간략하게 진행 하였다.

### `Github Action` 은 
1. 깃헙 레파지토리에서 발생하는 이벤트를 받아서 처리할 수 있게 인터페이스 제공
1. 레파지토리별 가상 서버를 깃헙에서 제공
    - 자체 호스팅 서버를 이용도 가능
1. 깃헙에서 제공하는 가상서버에는 해당 레파지토리에 대한 접근권한 설정이 미리 되어있는 상태  
`GITHUB_TOKEN` 을 이용한다고 한다.

이 정도의 정보를 갖고 행동으로 옮겼고, 삽질 끝에 배포 자동화 환경을 구축했다.

배포 과정을 요약하면 이렇다.  
1. github 저장소의 `work` 브랜치에 `push`를 한다.
1. 원격지에서 `빌드`를 수행한다.
1. `빌드` output을 github 저장소의 `master` 브랜치에 `push` 한다. (**배포완료**)

## Github Action 을 만들어보자

### Github Action 생성  
![image](https://user-images.githubusercontent.com/25237661/75607351-8b816d00-5b39-11ea-9a42-447925b85cbe.png)  

생성 버튼을 누르니 깃헙설정 경로에 `yml` 파일이 생성되는 걸 보니 **DSL** 의 느낌이 온다.
그래서 [Github Action Help](https://help.github.com/en/actions) 레퍼런스를 잠깐 읽었는데 머리도 아프고 역시 몸빵이 최고인 것 같아서 레퍼런스를 켜두고 일단 시작했다.

[Github Action Help](https://help.github.com/en/actions) 어디에선가 발췌한 Hello World 스크립트를 갖고 바로 몸빵을 시작하기로 결정했다.

**Hello World 스크립트**
``` yml
name: Greet Everyone
# This workflow is triggered on pushes to the repository.
on: [push]

jobs:
  build:
    # Job name is Greeting
    name: Greeting
    # This job runs on Linux
    runs-on: ubuntu-latest
    steps:
      # This step uses GitHub's hello-world-javascript-action: https://github.com/actions/hello-world-javascript-action
      - name: Hello world
        uses: actions/hello-world-javascript-action@v1
        with:
          who-to-greet: 'Mona the Octocat'
        id: hello
      # This step prints an output (time) from the previous step's action.
      - name: Echo the greeting's time
        run: echo 'The time was ${{ steps.hello.outputs.time }}.'
```

### Github Action 몸빵으로 배우기

깃헙 액션을 몸빵으로 체득하다 보면 커밋 히스토리가 지저분해질게 뻔히 보여서 일단 리모트 **브랜치를 분리**하는 작업을 먼저 진행 하였다.
> work-flow-test 브랜치 생성

분리된 브랜치에서 깃헙액션 DSL 을 생성하고, 사전에 찾아뒀던 Hello World 스크립트를 저장하였다.  
저장하는 순간 github 에 push 이벤트가 발생하였고, 아래와 같이 깃헙 액션 실행결과를 확인할 수 있었다.  
![image](https://user-images.githubusercontent.com/25237661/75607494-20389a80-5b3b-11ea-901c-e98d3204e967.png)

깃헙액션의 실행결과를 눈으로 확인하고 나서 바로 든 생각은 *`신기하네 약간 도커같은 느낌인걸?`* 이였다.

그리고 job 의 `step` 을 하나씩 단계별로 구성하면 내가 하려던 작업을 충분히 할 수 있겠다고 직감했고 깃헙 액션을 조금씩 채워나가기 시작했다.

1. 소스 체크아웃 `step` 만들기
이런저런 삽질을 해보다가 이정도는 이미 만들어져 있지 않을까 해서 찾아보니 [깃헙에서 제공](https://github.com/actions/checkout)하고 있어서 그걸 사용했다.  
1. ruby 설치하기 `step` 만들기
지킬 블로그는 ruby 패키지에 의존하여 동작하기 때문에 필수적으로 필요한 요소이다.  
이것도 [깃헙에서 제공](https://github.com/actions/setup-ruby)한다.
1. gem 설치하기 `step` 만들기  
지킬 블로그에 빌드에 필요한 gem 을 설치해야 한다. (내 로컬과 동일하게)  
여기서 삽질을 좀 했다.  
    1. `Gemfile.lock`이 존재하지 않아서 gem 설치가 순서대로 되지 않아서 오류 발생
    1. 젬 디펜던시 설치가 너무 오래 걸림 (액션이 돌때마다 새로 설치)  
    이에 대한 솔루션도 [깃헙에서 제공](https://github.com/actions/cache)하고 있어서 몇번 삽질 후에 성공적으로 적용하였다.
1. 블로그 빌드 `step` 만들기  
[기존에 프로덕션 배포시 사용하던 Rakefile을 일부 활용](https://github.com/actions/cache)하였다. `rake site:generate`
1. 블로그 배포 `step` 만들기  
쉘 스크립트 작성에 능숙치 못해서 여기에서 가장 많은 시간을 소요했다  
대충 요악하면 여기에서 진행하려고 하던 작업은 이렇다.
    1. 블로그 빌드 결과물을 임시 디렉토리로 이동
    1. 작업 브랜치를 `master` 브랜치로 이동
    1. 블로그 빌드 결과물 적용
    1. 변경사항 `commit`
    1. `commit` 내용을 github 저장소의 `master` 브랜치로 push

## 완성된 Github Action DSL

내가 삽질하면서 작성한 Github Action DSL 은 [여기서](https://github.com/JeHuiPark/JeHuiPark.github.io/pull/3) 확인할 수 있다.

- 2020-03-15 루비 모듈 캐싱안되는 현상 발견 및 [조치](https://github.com/JeHuiPark/JeHuiPark.github.io/pull/11)


## 깃헙액션 동작 결과

![image](https://user-images.githubusercontent.com/25237661/75608419-9391da80-5b42-11ea-87ed-498fc2f5a657.png)
