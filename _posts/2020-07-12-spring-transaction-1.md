---
layout: posts
title:  Spring Transactional 과 메소드 접근제어 수준
date:   2020-07-12 18:39:29 +0900
comments: true
categories: spring
---

* TOC
{:toc}

## Transactional 선언 메소드는 public 으로 지정하자
 `@Transactional` 어노테이션을 이용하여 선언적 트랜잭션 메소드를 작성할 때는 주의해야 할 점이 있다.  
메소드의 접근제어 수준을 `public` 으로 설정해야 `Spring` 에서 지원하는 트랜잭션을 획득 할 수 있다는 점인데, 이 스펙을 지키지 않으면 원하는 동작을 기대할 수 없을 것이다.

## 이유를 파헤쳐보자
스프링이 어플리케이션 컨텍스트를 구성하는 과정중에는 `@Transactional` 어노테이션을 스캔하여 트랜잭션 후보군을 탐색하는 과정이 존재한다.  

트랜잭션 후보군을 탐색하는 작업은 [`AbstractFallbackTransactionAttributeSource`][AbstractFallbackTransactionAttributeSource] 클래스를 상속받은 [`AnnotationTransactionAttributeSource`][AnnotationTransactionAttributeSource] 클래스가 담당하고 있다.

[`AbstractFallbackTransactionAttributeSource`][AbstractFallbackTransactionAttributeSource] 에서 트랜잭션 후보군 탐색 로직의 일부분을 확인해보자
![image](https://user-images.githubusercontent.com/25237661/87243130-5897a000-c46e-11ea-8d62-9092da18b680.png)
이 메소드는 `allowPublicMethodsOnly()` 값과 타겟 메소드의 `public` 여부에 따라 `null` 을 `fast - return` 하도록 작성되어 있다.   

`allowPublicMethodsOnly()` 메소드를 확인 해보자
![image](https://user-images.githubusercontent.com/25237661/87243854-694b1480-c474-11ea-8bff-6ba005c8e7bd.png)
기본 전략은 `false` 를 리턴하게 되어 있지만, 재구현을 가능성을 고려하여 설계된 메소드이다.

기본 구현체인 [`AnnotationTransactionAttributeSource`][AnnotationTransactionAttributeSource] 클래스를 확인해보자.
![image](https://user-images.githubusercontent.com/25237661/87243148-7402ab00-c46e-11ea-8870-442e89dc95ca.png)
설정변경이 가능하긴 하지만, 기본값은 `public` 메소드만 허용하는 것으로 되어있는 것을 확인할 수 있다.
 
기본 설정이 이렇기 때문에 `public` 메소드가 아니라면 `@Transactional` 어노테이션이 선언되어 있다고 하더라도, 메소드 접근제어 수준이 `public` 이 아니라면 트랜잭션을 획득하지 못한다.


[AbstractFallbackTransactionAttributeSource]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/interceptor/AbstractFallbackTransactionAttributeSource.html
[AnnotationTransactionAttributeSource]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/AnnotationTransactionAttributeSource.html
