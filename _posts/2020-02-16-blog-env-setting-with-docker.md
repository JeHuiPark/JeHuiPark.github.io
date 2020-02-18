---
layout: posts
title:  "GitHub 블로그 개발 환경을 Docker로 구성하자"
date:   2020-02-16 16:26:00 +0900
comments: true
categories: blog
tags: 
  - jekyll
toc: true
---

{:toc .toc__menu}

몇 가지 이유로 나는 이 블로그를 거의 1년간 방치하고 있었다. (~~*사실 핑계*~~)
- 2019년 06월 이직
- 지킬 블로그 개발환경 구성이 너무나 귀찮다
- 올바른 객체지향 설계를 위한 개념 잡기에 몰두

# 블로그를 방치했던 가장 큰 이유
개발환경 구성이 너무나 귀찮다는 점이다.

## 환경 구성이 귀찮았던 이유 
- 운영환경으로 부터 독립적이지 못함 (여기서 운영환경은 지킬서버가 돌아가는 환경을 말함 -> 개발환경)
- 깃헙에게 지킬 빌드를 위임하는 방식이 아닌 로컬 빌드 방식을 택함 (그 이유는 [여기에](https://jehuipark.github.io/blog/blog-publish))


# 내 블로그 살리자
일단 기존의 문제점은 내가 가장 잘 알고있는 상태이기 때문에 어떻게 살릴지에 대한 추상적은 계획은 세워두었다.

## 계획
아주 간단하다 도커를 활용하여 운영환경에서 분리하여 손쉽게 개발환경 구축이 가능하게 만들면 된다.  
다만, 나는 도커에 자신이 없다. (걸음마 수준)
> 그래서 나는 [이 글](https://www.44bits.io/ko/post/how-docker-image-work)을 참고하여 작업했다. (감사합니다 ❤️)

## 실행 (고군분투기)
1. 가장 먼저 jekyll 이미지를 땡겨 받자.    
`docker pull jekyll/jekyll`  
[도커허브](https://hub.docker.com)에서 jekyll을 검색하면 pull command를 확인할 수 있으며, 기초적인 레퍼런스도 확인이 가능하다.  

1. git에서 내 블로그 소스코드를 clone하자.  
`git clone 블라블라블라블라`

1. 터미널에서 내 블로그 소스코드 위치로 이동하자.  
![image](https://user-images.githubusercontent.com/25237661/74605023-56bbf180-5107-11ea-8605-3a9851342ce3.png)

1. 레퍼런스에 있는 커맨드를 그대로 일단 터미널에 때리자
``` sh
export JEKYLL_VERSION=3.8
docker run --rm \
  --volume="$PWD:/srv/jekyll" \
  -it jekyll/jekyll:$JEKYLL_VERSION \
  jekyll build
```
때리고 나서 확인해보니 현재 경로를 컨테이너의 `/srv/jekyll` 경로로 마운팅 시킨다는 옵션이 있는걸 확인했다. (~~제대로 했군ㅋ~~)  
명령어를 확인해보면 3.8 버전을 사용하도록 하고 있는데, 지킬 블로그 구성할 때 지킬 버전이 몇이였는지 기억이 안난다. 문제생기면 확인 하기로 하고 일단 수정하지 않기로 했다.  

  1. 오류가 난다... 역시 한번에 되면 섭하지  
  오류 내용은 이랬다. -> 대충 `지킬 빌드에 필요한 어떤 디펜던시가 존재하지 않는다` 라는 내용  
  ![image](https://user-images.githubusercontent.com/25237661/74605184-034aa300-5109-11ea-81dd-4cbe7e65a2c9.png)
  
  1. 처방전을 내리자.  
  *여기선 이렇게 간단하게 쓰지만 지킬을 너무 오랜만에 접해서 기억을 더듬느라 힘들었음*  
  과거의 내가 왜 그랬는지 기억이 나질 않지만 루비 모듈 의존성을 관리하는 Gemfile을 과거에 삭제해서 생긴 문제이다.  
      1. Gemfile을 일단 생성하자.  
      ```ruby
      source "https://rubygems.org"
      gem "jekyll-sitemap"
      ```

      1. 또 오류가 난다.  
      여기서 지킬 블로그 설정을 관리하는 파일을 확인 해야겠다는 생각이 들었다.   
      ![image](https://user-images.githubusercontent.com/25237661/74605336-a5b75600-510a-11ea-8fcd-f14aa01099e8.png)  
      오류 내용과 플러그인 리스트가 일치하는 거 보니 내 예상이 맞는듯 하다.

      1. Gemfile을 지킬 설정과 동기화 시키자.  
      ``` ruby
      source "https://rubygems.org"
      gem "jekyll-sitemap"
      gem "jekyll-mentions"
      gem "jekyll-paginate"
      gem "jekyll-seo-tag"
      gem "jekyll-gist"
      gem "jekyll-redirect-from"
      gem "jemoji"
      ```

  1. 이제 레퍼런스에 있는 커맨드를 그대로 일단 터미널에 때리자  
  오오오오 뭔가 되는듯 하다가 오류없이 작업이 종료 되었다. (엥?)  
  내가 원하는건 4000 포트로 로컬서버가 구동되는건데?  
  커맨드를 자세히 보니 컨테이너 구동완료후 실행할 명령어가 내가 원하는 명령어가 아니였다  
  레퍼런스 명령어는 `jekyll build`, **내가 원하는 명령어는 `jekyll serve`**
  
  1. 명령어를 수정하자 (1차)
  ``` sh
  docker run \
  -v "$PWD:/srv/jekyll" \
  -it jekyll/jekyll:3.8 \
  jekyll serve
  ```
  오 이번엔 뭔가 된 것 같다?  
  ![image](https://user-images.githubusercontent.com/25237661/74605502-e9f72600-510b-11ea-97ff-28c307d10a26.png)  
  **아니다 안된다**  
  ![image](https://user-images.githubusercontent.com/25237661/74605875-29734180-510f-11ea-9aeb-5a0e8bd001f5.png) 
  서버가 올라갔는데 `localhost:4000`으로 접속을 못하는 이유는 컨테이너와 호스트 포트 바인딩 문제라고 판단했다.

  1. 명령어를 수정하자 (2차)  
  ``` sh
  docker run \
  -v "$PWD:/srv/jekyll" \
  -p 4000:4000 \
  -it jekyll/jekyll:3.8 \
  jekyll serve
  ```
  **잘 된다**  
  ![image](https://user-images.githubusercontent.com/25237661/74605870-1c565280-510f-11ea-934d-2c695bfc4746.png)  
  잘 되지만 지금 상태에서는 **문제점**이 있다.  
  docker run을 할 때마다 루비 모듈을 처음부터 내려 받느라 엄청난 시간을 소요함  
  도커허브의 jekyll 이미지 레퍼런스를 확인하니 [이런내용](https://github.com/envygeeks/jekyll-docker/blob/master/README.md#caching)이 존재했다.  
  대충 `컨테이너의 루비 모듈 설치경로와 호스트의 볼륨을 마운팅 시켜라`라는 내용  
  > docker run할 때 컨테이너 이름을 지정하여 docker start와 stop을 이용하는 방법도 존재하며,  
  이 방법은 컨테이너를 새로 생성할 경우에 루비 모듈을 처음부터 내려 받아야 한다

  1. 명령어를 수정하자 (3차)  
  ```sh
  docker run \
  -v "$PWD:/srv/jekyll" \
  -v "$PWD/.cache/bundle:/usr/local/bundle" \
  -p 4000:4000 \
  --name blog \
  -it jekyll/jekyll:3.8 
  jekyll serve
  ```
  컨테이너의 루비 모듈 설치경로와 호스트 경로의 볼륨을 마운팅 하라는 옵션을 추가하고, 컨테이너 이름을 blog로 지정했다.  
  추가로 루비 모듈 캐시경로를 git에서 추적하지 않도록 설정을 추가했다.  
  이제 컨테이너를 지우고 다시 생성해도 `.cache` 디렉토리에 미리 설치된 루비 모듈이 존재하니, **한번이라도 환경구성을 했다면** 컨테이너 구동이 이전보다는 굉장히 빨라졌다.

  1. docekr-compose를 작성하자  
  최초 환경 구동시에 더 편하게 환경울 구성하기 위해 `docker-compose.yml`을 작성했다.  

## 보너스  
최초 환경 구동시에 더 편하게 환경울 구성하기 위해 `docker-compose.yml`을 작성했다.  
```yml
version: '3.7'
services:
  jekyll:
    container_name: blog
    image: jekyll/jekyll:3.8
    ports:
      - 4000:4000
    volumes:
      - $PWD:/srv/jekyll
      - $PWD/.cache/bundle:/usr/local/bundle
    command: jekyll serve
```

# 마치며
이 작업으로 나는 이제 `git clone 블라블라블라` 명령어와 도커를 이용하여 블로그 개발환경을 구성할 수 있게 되었다.  
하지만 아직도 개선할 점은 남아있다.
  
## 앞으로 할 일
  - CD 구성하기  
  이건 과거에 윈도우 환경에서 [배포자동화](https://jehuipark.github.io/blog/blog-publish)를 위해 사용하던 Rakefile을 참고하면 될 듯 하다.