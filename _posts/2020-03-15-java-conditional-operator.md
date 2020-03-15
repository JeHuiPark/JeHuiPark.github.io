---
layout: posts
title:  "3항 연산자와 Unboxing 그리고 NullPointException"
date:   2020-03-15 15:00:00 +0900
comments: true
categories: java
tags: 
  - java
---

* TOC
{:toc}


## 떡밥

얼마전에 팀 내에서 재밌는 이슈가 나왔다.  
대충 아래와 같은 방식으로 리턴하는 메소드가 존재하였는데.  
``` java
Integer exampleMethod() {
  return booleanExpression ? primitiveValue : integerObj;
}
```

`booleanExpression` 의 값이 `false` 일 때, `integerObj` 의 값이 `null` 이면 `NullPointException` 오류가 터지는 문제였다.

언뜻 보면 `booleanExpression` 의 값이 `false` 이면, `integerObj` 의 값이 무엇이던 그대로 반환할 것 같은 메소드이지만, 알고보면 그렇지 않다.

이 오류를 재현하기 위해 간단한 테스트를 작성하고, 실행 해보자
``` java
@Test
public void ex1() {
  try {
    Integer a = null;
    Integer b = false ? 0 : a;
    Assert.fail();
  } catch (NullPointerException expected) {
    System.out.println("예상되는 에러");
  }
}
 ```

 **테스트 결과**  
![image](https://user-images.githubusercontent.com/25237661/76698396-bb0da900-66e5-11ea-9b27-9ef98bcbdacd.png)

### 왜 `NullPointException` ?

결론부터 말하면 `java` 의 `unboxing` 과 관련된 이슈였다.

요악하면 변수 `a` 를 `unboxing` 하는 과정에서 `NullPointerException` 이 발생한다.

#### 어떻게 알아내었는가

머릿속에 여러 생각이 들었다.

- `이펙티브 자바` 책의 한 구절이 떠올랐다. 
    > 컴파일러로 인해 개발자가 작성한 코드가 예상과 다르게 동작할 수 있다.
- `NullPointerException` 예외가 발생할 수 있는 포인트가 3항 연산자 이외에는 보이지 않는다.
- 3항 연산자의 2항의 반환 타입이 `primitive` 인게 걸린다.
- 혹시 `null` 로 초기화 된 `a` 를 언박싱 하려고 하니?
- 만약 2항의 값이 `boxing object` 이면 에러없이 `null` 을 반환할까?

나름의 정보를 종합하여, 한가지 가설을 세우고 바로 테스트를 작성해 보았다.  
``` java
@Test
public void ex2() {
  Integer a = null;
  Integer b = false ? (Integer) 0 : a;
  Assert.assertNull(b);
}
```

**테스트 결과**  
![image](https://user-images.githubusercontent.com/25237661/76698556-bfd35c80-66e7-11ea-84cb-b4754df909f0.png)

**내가 세운 가설**은 이랬다.  
- 2항의 값이 `primitive` 이기 때문에, 3항의 `boxing object`가 `unboxing`이 되는 것은 아닐까?

이렇게 나는 테스트 코드를 통해 이번 3항 연산자 떡밥은 `unboxing` 과 관련이 있다는 것이 증명 되었다고 판단하였다.

### `unboxing` 의 동작방식이 궁금해

`boxing` 이라던지 `unboxing` 키워드는 정말 지겹도록 들어봤을 것이며, 이것을 설명하는 것은 주제를 넘어가니 생략한다.

당시에, 나는 2가지의 키워드가 의미하는 바가 무엇인지는 알고 있었지만, 솔직히 말하면 실제로 어떻게 동작하는지에 대해서는 구체적으로 알고 있는 상태는 아니였다.

이때의 계기로 나는 `unboxing` 이 어떻게 동작하는지 알아보고 싶어져서 `바이트 코드` 를 읽어보기로 결정하였다.

![image](https://user-images.githubusercontent.com/25237661/76698970-c5cb3c80-66eb-11ea-8627-85a41970551e.png)

이미지에 보이는 자바코드 두줄은 바이트 코드로 이렇게 표현되고 있었다.
``` java
L0
 LINENUMBER 12 L0
 ACONST_NULL
 ASTORE 1
L3
 LINENUMBER 13 L3
 ALOAD 1
 INVOKEVIRTUAL java/lang/Integer.intValue ()I
 INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;
 ASTORE 2
```
천천히 순서대로 읽어보자. ([오라클 문서](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html#jvms-6.5.astore)와 같이 보면 도움이 된다.)
1. `LO` 은 자바코드 12번 라인에 대응하는 코드이다. 
    1. `ASTORE 1` -> `null` 이라는 상수를 `로컬 참조변수 저장소`의 `1`에 저장한다. 
2. `L3` 은 자바코드 13번 라인에 대응하는 코드이다.
    1. `ALOAD 1` -> `로컬 참조변수 저장소`의 `1`에서 객체를 로드 해라. (`null` 예상)
    1. `INVOKEVIRTUAL java/lang/Integer.intValue ()I` -> 로드한 객체를 이용하여 `Integer` 클래스의 `intValue` 메소드를 실행해라. (**`null` 참조 상태이므로 `intValue()` 메소드를 실행하면 `NullPointException` 이 발생**한다.)
    1. `INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer` -> `Integer` 클래스의 `valueOf`라는 정적 메소드를 실행하면서 I를 넘겨라. 정적 메소드의 반환 타입은 `java/lang/Integer` 이다.
    1. `ASTORE 2` -> 반환값을 `로컬 참조변수 저장소`의 `2`에 저장하라

> 바이트 코드에서 보면 13번 라인의 3항 연산자가 삭제된 것을 확인할 수 있는데, 이것은 1항의 값이 컴파일 시점에 이미 결정되어 있기 때문에 실행시점에는 2항의 값이 쓸모없는 값이 되어 버린다 그렇기 때문에, 컴파일러가 최적화한 결과라고 생각하면 되며, 이 글에서 중요하지 않는 부분이다.

바이트 코드를 읽어보니 언박싱이 어떻게 동작하는 건지 알 수 있을것 같다.  
바이트 코드 분석중 2-2 를 확인해보면 `INVOKEVIRTUAL java/lang/Integer.intValue ()I` 이 부분이 박싱 객체를 언박싱하는 과정으로 보이는데, `null` 참조를 이용하여 `intValue()` 메소드를 실행하려고 시도하니 `NullPointException` 이 발생한 것이였다.

## 1짤 요약  
![image](https://user-images.githubusercontent.com/25237661/76700066-e9e04b00-66f6-11ea-9263-1f4f049917a3.png)


## 후기
퇴근 직전에 재밌는 떡밥이였다.
- 이 이슈는 알고보니 아주 유명한 이슈라고 한다. 
    - 이것도 자바 스펙중 하나로 오라클에 [오피셜 문서](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.25)가 존재한다.
- 3항 연산자에 박싱 객체와 언박싱 객체가 공존할 때, 언박싱 객체를 박싱객체로 변환하는 방법도 있을법 하지만, 컴파일러는 언박싱 처리를 하도록 하고 있다. 이유가 뭘까?
    - 그냥 단순히 객체 생성 비용을 아끼기 위해서 일까? 다른 이유가 있을까?