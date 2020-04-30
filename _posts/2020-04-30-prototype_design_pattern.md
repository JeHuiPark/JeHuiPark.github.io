---
layout: posts
title:  "디자인 패턴 : Prototype Design Pattern"
date:   2020-04-30 20:00:00 +0900
comments: true
categories: note
---

* TOC
{:toc}

## 정의
원본 객체를 사용하여 새로운 사본 객체를 생성하는 패턴  
> 복사 객체의 타입은 서브 클래스에서 정의

## 효과
- 클라이언트에서 같은 상태의 객체를 생성하기 위해 알아야 할 정보가 줄어든다.
- 팩토리 메소드와 다르게 객체 생성에 대한 책임을 클라이언트에게 일부 위임하기 때문에 인스턴스를 쉽게 다양화 시킬 수 있다.
    >서브 클래싱 감소
- 클래스의 인스턴스들이 서로 다른 상태 조합중 하나일 때 유용하다.
    >프로토 타입 인스턴스를 기반으로 동적으로 클래스에 따라 설정을 변경하여 사용

## 구조
[예제코드](https://github.com/JeHuiPark/java-sample/tree/master/design-pattern/src/main/java/com/example/jehuipark/prototype_pattern)  
[예제코드 테스트](https://github.com/JeHuiPark/java-sample/blob/master/design-pattern/src/test/java/com/example/jehuipark/prototype_pattern/ClientTest.java)  
![image](https://user-images.githubusercontent.com/25237661/80703013-f5f46080-8b1c-11ea-9dc0-f8b4d0aa94b1.png){:width="500px"}  

## 적용예시
Java 에서 json 처리를 위해 자주 사용되는 라이브러리 중 `Jackson` 라이브러리를 살펴보면 원형패턴이 적용된 예시가 있다.

![image](https://user-images.githubusercontent.com/25237661/80708450-a4e96a00-8b26-11ea-9cc1-72b14688e92f.png){:width="500px"}  
![image](https://user-images.githubusercontent.com/25237661/80704003-c2b2d100-8b1e-11ea-95ac-491969cb41d4.png){:width="500px"}

`ObjectMapper` 에서 의존하는 몇몇 추상화 클래스들은 구현체에게 `copy` 메소드를 구현할 수 있도록 하여, `ObjectMapper` 의 `copy` 메소드를 지원한다.

## 고려 사항
- 깊은 복사와 얕은 복사 타겟 설정
    > 보통 상태머신 객체는 깊은 복사를, `stateless` 객체는 얕은 복사를 하면 될 듯 하다
- 복사를 지원하지 않는 객체를 다수 포함하고 있다면, 복사기능을 구현하기 힘들 수 있다.

## 참고자료 
> *GoF의 디자인 패턴 p169 - p180*
