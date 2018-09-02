---
layout: post
title:  "GoogleAnalytics Report API 연계"
date:   2018-09-02 17:50:21 +0900
comments: true
categories: GoogleAnalytics
---

GoogleAnalytics를 들어보셨나요.
GoogleAnalytics는 개발자가 약간의 노력만으로 특정 서비스를 이용하는 사용자들의 활동을 추적할 수 있게 도와주는 아주 유용한 도구입니다.
물론 추적 데이터 또한 구글 자체서버에 저장되고 있으니 기본적인 기능만 이용할꺼라면 개발자는 정말 아무것도 할 게 없게 만들어주는 강력한 도구이죠.
이러한 추적데이터를 [GoogleAnalytics 공홈](https://analytics.google.com/analytics)에서 확인 할 수도 있지만 구글은 고맙게도 추적데이터를 개발자가 직접 서비스 할 수 있게 **GoogleAnalytics Report API** 서비스를 제공하고 있습니다.

이번 글에서는 바로 java로 개발된 서버에서 **GoogleAnalytics Report API** 를 어떻게 연계하는지에 대해 공유해보겠습니다.

#### 시작전에 GoogleAnalytics를 어느정도는 알고 있고 사용중이라는 가정을 하고 진행함을 미리 알려드립니다.


우선 Google Play Console페이지로 이동하여 아래와 같은 순서로 프로젝트를 생성합니다.
(*GoogleAnalytics서비스와 GoogleAnalytics Report API는 다른 서비스이며 GoogleAnalytics Report API는 GoogleAnalytics의 데이터를 조회하기 위한 인터페이스*)
![이미지삽입](https://user-images.githubusercontent.com/25237661/44958518-6d26bb00-af1c-11e8-8c9d-86c173c277e6.png)


프로젝트 생성 후
좌측에 라이브러리 메뉴에 들어가서 **Google Analytics Reporting API** 를 찾아서 사용함으로 설정합니다.

다음으로 GoogleAnalytics Reporting API 사용을 위한 API 인증정보를 만들어주어야 합니다.
**추후 Google Analytics 콘솔에서 데이터조회 권한을 부여할 때 사용되는점 미리 알아주세요.**
아래와 같이 서비스 계정관리를 클릭하여 프로젝트 서비스계정을 발급하는 페이지로 이동하여 서비스 계정 발급
(_발급받는 과정중 마지막에 **json파일을 제공해주는데 반드시! 잘! 갖고 있어야 합니다.**_)
![이미지삽입](https://user-images.githubusercontent.com/25237661/44958519-6d26bb00-af1c-11e8-9bff-b4844d6fffd9.png)

서비스 계정을 발급받으면 생성되는 이메일 주소를 복사합니다.
![이미지삽입](https://user-images.githubusercontent.com/25237661/44958520-6d26bb00-af1c-11e8-8419-ed4c76bc35f4.png)

아래와 같이 GoogleAnalytics콘솔로 돌아가서 데이터조회 권한을 서비스 계정에 부여합니다.
![이미지삽입](https://user-images.githubusercontent.com/25237661/44958522-6dbf5180-af1c-11e8-9c06-08fc9ba05087.png)

여기서 여기서 GoogleAnalytics 와 GoogleAnalytics Report API연계는 마무리가 됩니다.

실제 소스레벨에서 GoogleAnalytics Report API를 어떻게 연계하고 있는지는 다른 글에서 알아보겠다는 것을 알려드리며 마지막으로 GoogleAnalytics Report API샘플 소스를 구동한 모습을 공유해드리면서 이번 글을 마무리 짓겠습니다.

![ga_report_api_07](https://user-images.githubusercontent.com/25237661/44958517-6d26bb00-af1c-11e8-9495-b1ac8185c9af.png)
