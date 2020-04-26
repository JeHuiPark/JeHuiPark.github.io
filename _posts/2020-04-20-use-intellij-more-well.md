---
layout: posts
title:  "IntelliJ 잘 쓰기 Tip"
date:   2020-04-20 23:00:00 +0900
comments: true
categories: note
---

* TOC
{:toc}

## No newline at end of file
`intelliJ` 에서는 `커맨드` + `N` 단축키를 이용하여 테스트 클래스를 자동생성할 수 있는데, 이 기능을 그냥 이용하게 되면 거슬리는게 하나 있다.  
![test_gen](https://user-images.githubusercontent.com/25237661/79998744-0b9ed000-84f6-11ea-845c-b0e5f621453b.gif){:width="500px"}   

**소스트리**  
![image](https://user-images.githubusercontent.com/25237661/80000609-4f92d480-84f8-11ea-81b9-685007f3347f.png){:width="500px"}  

**깃헙**  
![image](https://user-images.githubusercontent.com/25237661/80001543-73a2e580-84f9-11ea-9614-1f321e55f53a.png){:width="500px"}  

**터미널**  
![image](https://user-images.githubusercontent.com/25237661/79999509-f37b8080-84f6-11ea-9b31-2604fd811578.png){:width="500px"}  

EOF 전에 공백 라인이 없어서 생기는 문제인데, **마지막 라인에 공백라인 하나만 추가**하면 해결된다.

그런데 테스트 코드를 생성할 때 마다 공백라인을 추가하는 행위를 하자니 너무 원시적이며, 되게 자주 까먹는다.
> PR 2번중에 1번꼴로 하는 실수였다.
이렇게 실수를 자주하다가 현타가와서 도구를 활용하기로 했다.

더블 쉬프트 후 `file and code templates` 입력  
![image](https://user-images.githubusercontent.com/25237661/80002548-bfa25a00-84fa-11ea-866a-9af093ce17d1.png){:width="500px"}  

여기에서 자주 사용하는 테스트 도구를 선택  
![image](https://user-images.githubusercontent.com/25237661/80002583-caf58580-84fa-11ea-8bc9-5889d91a3471.png){:width="500px"}  

공백 라인을 추가한다.  
![image](https://user-images.githubusercontent.com/25237661/80002993-596a0700-84fb-11ea-9173-b05e93c795cf.png){:width="500px"}  
**끝!**

## 보일러 플레이트 해결하기
코딩을 하다보면 보일러 플레이트가 상당히 많이 발생하는데 툴에서 지원하는 기능을 적절히 이용하면, 좀 더 스마트하게 작업을 할 수 있다.

### LiveTemplate
Example Java Test Code boilerplate  
`Preferences > Editor > Live Templates`  

![image](https://user-images.githubusercontent.com/25237661/80306715-e06ef600-87ff-11ea-9cd5-541f0d5694a6.png){:width="500px"}  

#### 템플릿 만들기
![image](https://user-images.githubusercontent.com/25237661/80306796-668b3c80-8800-11ea-8a18-d0f53e5b3f51.png){:width="500px"}
- `expression` 에서는 이런식으로 `groovy` 문법을 지원한다.  
    ``` groovy
    groovyScript("_editor.getDocument().getText().split(\"@.*Test\").length-1")
    ```

#### 템플릿 적용위치 설정하기
![image](https://user-images.githubusercontent.com/25237661/80307282-fd58f880-8802-11ea-87ef-295d50537179.png){:width="300px"}  
- Java 파일에서 선언문 작성시에 라이브템플릿을 활성화 한다.

#### 사용 예시
![live_template](https://user-images.githubusercontent.com/25237661/80307461-275eea80-8804-11ea-8634-69ea9ff0304c.gif){:width="500px"}  





### FileTemplate
Example Vue boilerplate  
뷰로 프론트 작업을 한적이 있는데, 보일러플레이트 코드가 상당히 많아서 파일 템플릿을 적절하게 수정하여 작업 효율성을 높였다.  
`Preferences > Editor > file and code templates`  
![image](https://user-images.githubusercontent.com/25237661/80306590-34c5a600-87ff-11ea-81be-af466056dfc0.png){:width="500px"}


## 터미널에서 Intellij 실행하기
작업을 하다보면 **터미널에서 intellij 를 실행하고 싶을 때**가 자주있다.  
그럴땐 이런 설정을 이용하면 된다.  
![image](https://user-images.githubusercontent.com/25237661/79994673-3d616800-84f1-11ea-9bc7-60151edc62ee.png){:width="300px"}  
![idea_open_in_terminal](https://user-images.githubusercontent.com/25237661/79996611-9c27e100-84f3-11ea-81c2-c80c3b7c8152.gif)

