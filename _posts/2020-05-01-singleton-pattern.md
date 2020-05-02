---
layout: posts
title:  "디자인 패턴 : Singleton"
date:   2020-05-01 20:00:00 +0900
comments: true
categories: study
tags: 
  - 디자인 패턴
---

* TOC
{:toc}

## 정의
클래스에 대한 인스턴스를 하나로 제한하는 패턴

## 특징
- 인스턴스의 개수가 1개임을 보장
- access point 제한
- 유일한 인스턴스이면서 서브 클래싱으로 확장을 지원

## 예제코드
생성자를 은닉시키고, 객체 생성을 제한하여 인스턴스가 1개임을 보장하는 것에 초점을 맞춘다.

**간단한 싱글톤**  
`thread-safe` 요구사항이 없다면 아래와 같은 구현으로 충분하다.  
```java
final class BasicSingleton {
  private static BasicSingleton instance;

  private BasicSingleton() {};

  // thread non safe
  public static BasicSingleton getInstance() {
    if (instance == null) {
      instance = new BasicSingleton();
    }
    return instance;
  }
}
```
<br>

**안전한 싱글톤**  
`thread safe` 요구사항이 있다면 생성자 구현에 좀현 더 신경을 써야한다.
```java
final class ThreadSafeSingleton {

  private static ThreadSafeSingleton instance;

  private ThreadSafeSingleton() {}

  public static ThreadSafeSingleton getInstance() {
    if (instance == null) {
      safetyInitialize();
    }
    return instance;
  }

  private static final Object _lock = new Object();
  private static void safetyInitialize() {
    synchronized (_lock) {
      if (instance == null) {
        instance = new ThreadSafeSingleton();
      }
    }
  }
}
```
<br>

**클래스 로더를 활용한 방법**  
객체 생성을 `inner class` 에게 위임하여 초기화 시점을 제어  
```java
final class ThreadSafeSingleton2 {

  public static ThreadSafeSingleton2 getInstance() {
    return Inner.INSTANCE;
  }

  private static final class Inner {
    static final ThreadSafeSingleton2 INSTANCE = new ThreadSafeSingleton2();
  }
}
```
<br>

**서브클래싱을 지원하는 싱글톤**  
```java
abstract class SuperSingleton {

  private static final InstanceRegistry INSTANCE_REGISTRY = new InstanceRegistry();

  protected SuperSingleton() {}

  protected static void register(String name, SuperSingleton singleton) {
    INSTANCE_REGISTRY.addRegistry(name, singleton);
  }

  public static SuperSingleton getInstance() {
    SuperSingleton lookupInstance = defaultLookup();
    if (lookupInstance == null) {
      throw new IllegalStateException();
    }
    return lookupInstance;
  }

  private static SuperSingleton defaultLookup() {
    return lookup("default");
  }

  public static SuperSingleton lookup(String name) {
    return INSTANCE_REGISTRY.lookup(name);
  }

  private static class InstanceRegistry {
    private static final Map<String, SuperSingleton> REGISTRY = new HashMap<>();

    void addRegistry(String name, SuperSingleton instance) {
      if (REGISTRY.get(name) != null) {
        throw new IllegalStateException();
      }
      REGISTRY.put(name, instance);
    }

    SuperSingleton lookup(String name) {
      return REGISTRY.get(name);
    }
  }
}
```

[예제코드](https://github.com/JeHuiPark/java-sample/tree/master/design-pattern/src/main/java/com/example/jehuipark/singleton)  
[예제코드 테스트](https://github.com/JeHuiPark/java-sample/tree/master/design-pattern/src/test/java/com/example/jehuipark/singleton)

## 참고자료 
> *GoF의 디자인 패턴 p181 - p190*
