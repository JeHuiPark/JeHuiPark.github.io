---
layout: posts
title:  오브젝트
date:   2019-07-22 00:00:00 +0900
comments: true
categories: study
---

오브젝트

책을 보기위한 기본소양을 갖추기 위하여 아래와 같은 내용을 소개
개발자들은 코딩을 왜 할까? motivation = 돈
왜 그렇게 짰어? 돈이 덜드니까.. 이렇게 하면 돈을 버니까 왜 ???

## Philosophy
철학에 관한 이야기

켄트백은 생각하는 코드의 틀로 3가지를 제안
  - Value 가치
    - Communication 소통하기 쉬운 코드
    - Simplicity 간단한 코드
    - Flexibility 유연한 코드
  - Principle 원칙 - 예외적인 상황을 즉각적으로 파악할 수 있게 한다.
    - Local consequenes 생명주기는 되도록이면 짧게
    - Minimize repetition 중복은 최소화 (중복은 발견되는 것)
    - Symmetry 짝을 맞추어라
    - Convention
  - Pattern 패턴 (가치와 원칙을 베이스로 하는 반복되는 유형 )
Xoriented // 이게뭐지? 어떠한 유형의 사고의 틀?
OOP: SOLID, DRY
Functional,
Reactive,
...

Q. 강의에서 아래 내용은 왜 나온것일까??
  - Relativism: 토마스 쿤 (과학혁명의 구조)
  과학은 절대적인 진리가 아닌 상대적이다.  
  현재 우리가 알고 있는 과학은 어떻게 보면 종교와 비슷하다.

  - Rationalism: 러커토시 임레(수학적 발견의 논리: 증명과 반박)
  과학은 합리적이다라며 근거를 제시

A. 합리주의를 통해 기준을 도출하고 기준점을 토대로 상대성을 갖는다
상대주의와 합리주의를 공존, 사고방식을 넓혀라

시시각각 변화하는 요구사항을 계속 수렴하면서 납기일을 지켰다.
  어떻게? 그게 바로 이책의 주제
유연성, 격리성, 견고성

변화에 따른 격리, 어떻게?
현재 객체지향 방법론에서 도메인을 격리시킬 유일한 수단은 역활모델
역활모델을 이해하고 OOP 세계에 입문해야 한다
역활모델 설계를 잘하려면? 추상화를 잘 해야한다.

## Abstraction
Generalization: 일반화 - modeling, function, algorithm
Association: 연관화 - reference, dependence
Aggregation: 집단화 - group, category

데이터 추상화
  모델링: 어떤 목적에 맞추어서 필요한 정보를 추려내는 작업
  카테고리 분류
  공통분모

절차(함수) 추상화
  일반화
    함수의 개수가 줄어든다 << 공통점을 찾아서 인터페이스를 추출

  캡슐화 (캡슐화에 대한 정의는 여러가지가 존재)
    캡슐화 개념이 왜 나왔을까?
      캡슐화에 실패한다면 복잡성이 노출되며 사용자의 지능이 높아져야한다. >> 휴먼에러가 높아진다, (사용성이 낮아진다)
      캡슐화에 성공한다면 사용자의 사용성이 높아진다. >> 휴먼에러가 낮아진다 (사용성이 높아진다)

객체 추상화
  Generalization
  Realization
  Dependency
  Association
  ..

객체지향이 어려운 이유는?? 실제로 코드레벨에 이러한 개념을 실제로 적용하는건 상당히 어렵다
이해하고 코드레벨에 적용이 된다면 유연성, 격리성, 견고성을 확보하게 될 것이다.

## Program & Timing
프로그램 생명주기? (스크립트는 생략)
  LANGUAGE CODE // LINT TIME
  MACHINE LANGUAGE // COMPILE TIME
  FILE
  LOAD
  RUN  // RUN TIME
  TERMINATE

폰 노이만 머신 (동기화 명령)
메모리 영역에 명령이 적재된 순서대로 실행
Runtime
  Loading
    메모리 명령세트와, 값세트가 로드
  Instruction Fetch & Decoding
    CPU 제어유닛(디코더), 연산유닛(제어정보), 데이터유닛(메모리 계수기) << 외부버스
    디코더는 메모리영역에 있는 명령을 페칭 후 디코딩 (메모리 영역에 있는 명령을 CPU가 이해할 수 있는 명령으로 변경하는 단계)
    그리고 연산유닛으로 명령을 전달
  execution
    연산유닛은 명령 실행을 위한 값을 데이터유닛으로 부터 전달 받고 실행후 결과값 데이터유닛에게 전달, 데이터유닛은 메모리의 특정영역에 값을 전달

loading 과정을 좀 더 디테일하게 (언어마다 상이한점은 존재하지만 유사)
  essential definition loading 프로그램을 구동하기 위한 기초적인 정의부터 로딩 (컴파일러가 처리)
  vtable mapping 변수와 메모리가 매핑될 수 있는 이유 >> 가상 메모리와 실제 메모리를 매핑하는 표를 생성
  run
  runtime definition loading 자바의 클래스로더를 생각하면 된다. (c는 해당 개념이 존재하지 않는다)
  run

Runtime은 상대적이다, 유연하다, 애매모호

## Pointer of Pointer
특정 포인터를 직접 가리키지 않고, 어떤 값을 얻기 위해서 포인터를 찾은 후 그 포인터를 보고 포인터를 다시 찾는다
a = "TEST"  // a의 주소는 11이라고 가정
b = &a      // b는 a의 주소인 11이라는 주소값을 갖겠지? 포인터b는 "TEST"가 될것이고
c = b, d = b // c와 d도 b가 가리키고 있는 a의 주소를 갖게 될꺼야
k = "ABC"   // k의 주소가 28이라고 가정
b = &k      // 여기부터 b는 k의 주소인 28을 가리키게 될꺼야

여기서 발생하는 문제는????
c와 d는 b와 같은것이라고 착각하게 만든다 (직접참조의 문제) c,d != b

b = {a: &b, v:3} // 참조에 참조 개념 (간접참조)
c = b, d = b
b.a = &k

이러한 개념은 interface에 활용된다고 함

# Value & Identifier
객체지향에서 객체의 동질성을 평가하는 방법은? 객체의 식별자

# Polymoriphism
Substituion 대체가능성
Internal identity 내적동질성

대체 가능성에 대한 설명
객체의 다양성은 공짜가 아니다.
그만큼의 연산이 추가된다
```
class Worker implements Runnable {
  @Override public void run(){
    //..
  }
}
Runnable worker = new Worker();
worker.run();
```
해당 코드가 동작되는 실제 과정은 이렇다.
Runnable이라는 인터페이스 안에 있는 포인터안에서 run이라는 포인터를 찾고, 그대로 쓸 수없으니
구현체의 메소드의 포인터를 찾은 후 실행된다.
이러한 과정은 컴파일 타임에 개별 포인터 참조 구조체들을 데피니션 타임에 로딩 (이것은 C++베이스 코드 기반)

내적 동질성을 설명하는 코드
```
class Worker implements Runnable {
  @Override public void run(){
    //..
  }
  public void print(){
    run();
  }
}
class HardWorker extends Worker {
  @Override void run(){
    // ...
  }
}
Runnable worker = new HardWorker();
worker.print();
```
