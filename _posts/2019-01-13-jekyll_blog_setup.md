---
layout: posts
title:  "윈도우에서 지킬(jekyll)블로그 환경 구축"
date:   2019-01-13 17:50:21 +0900
comments: true
categories: blog
tags:
  - jekyll
---
기존에 사용하던 블로그 테마가 맘에들지 않아 테마변경을 진행하기로 결정하였다.
현재 사용중인 장비가 예전에 블로그를 운영하던 장비가 아니라 새로 환경을 구축했어야 됬는데
오랜만에 하려니 많은 시간이 소요되어 추후에 시간절약을 하기위해 메모한다.

1. **Ruby 설치  (지킬 패키지를 개발한 언어)**

    - [다운로드주소](https://rubyinstaller.org/downloads/)
    - 루비와 루비 개발자킷을 같이 다운받는다 (루비로 개발된 패키지들은 버전 디펜던시 이슈가 상당히 많기때문에 2.3.3 버전을 권장한다.)


    ![ruby_v](https://user-images.githubusercontent.com/25237661/51083514-dd610b00-175e-11e9-8455-8ab0e72547de.jpg)
    ![ruby_dev](https://user-images.githubusercontent.com/25237661/51083512-d934ed80-175e-11e9-8bd0-d078c86b40a4.jpg)

    - 루비 설치


    설치과정에서 __Add Ruby executables to your PATH__ 를 꼭 체크하여 환경변수를 등록

    - devkit 설치


    원하는 경로에 devkit을 압축해제 한 후 CMD를 실행하여 해당경로에서 아래와 같은 명령어를 입력합니다.
      ```cmd
      ruby dk.rb init
      ruby dk.rb install
      ```

1. **Jekyll 설치**

    - 커맨드 라인에 다음과 같이 입력합니다.
    ``` cmd
    gem install jekyll bundler github-pages
    ```
    위와 같은 명령어는 지킬 패키지를 설치할때 깃헙에서 사용하는 gem패키지도 함께 설치하도록 합니다.
    깃헙 gem dependency는 [여기서](https://pages.github.com/versions/) 확인하세요


1. **블로그 테마 선택**

    [참고사이트](http://jekyllthemes.org/)

1. **블로그 테마 설정**

    \_config.yml 수정

1. **지킬 블로그 실행 (웹서버 구동)**

    블로그 테마가 설치된 디렉토리로 이동후 커맨드라인에 다음과 같이 입력합니다.

    ```cmd
    jekyll serve
    ```
    ps. 인코딩 타입이 UTF-8 환경에서 작업을 하면 아래와 같은 **Invalid CP949 character** 에러를 볼 수 있습니다.   

    ![ruby_encode_error](https://user-images.githubusercontent.com/25237661/51084295-8a8d5080-176a-11e9-89d7-73ecff240c78.jpg)

    이때 당황하지 않고 커맨드라인에 아래와 같이 입력하여 UTF-8 환경으로 바꿔주시면 됩니다.
    ```cmd
    chcp 65001
    jekyll serve
    ```

    정상적으로 구동이 됬다면 아래와 같은 메시지를 확인할 수 있습니다.
    ![jekyll_serve](https://user-images.githubusercontent.com/25237661/51084348-39319100-176b-11e9-8322-1c0f06750617.jpg)

    메시지를 확인해보면 Destination속성이 있는데 이것은 지킬패키지에서 소스파일을 빌드하여 정적인 파일로 변환하는곳이라고 생각하면 됩니다. 즉 실제로 클라이언트가 웹상에서 조회하는 페이지가 됩니다.


    실제로 \_site 디렉터리에는 빌드되기전의 페이지조각들이 모여 완성된 정적페이지(html)로 저장되어 있는것을 확인할 수 있습니다.

1. **페이지 확인**

    ![page](https://user-images.githubusercontent.com/25237661/51084405-8b26e680-176c-11e9-8837-3b85e87f40d4.jpg)
