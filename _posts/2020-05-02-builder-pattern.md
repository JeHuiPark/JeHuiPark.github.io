---
layout: posts
title:  "디자인 패턴 : Builder Pattern"
date:   2020-05-02 20:00:00 +0900
comments: true
categories: study
tags: 
  - 디자인 패턴
---

* TOC
{:toc}

## 정의
복잡한 객체를 생성하는 방법과 표현하는 방법을 정의하는 클래스를 분리하는 패턴

## 특징
- 빌더에 의해 객체 생성방법이 추상화 되기 때문에 객체 생성 난이도가 낮아지며, 객체 생성방법이 클라이언트로 부터 어느정도 자유로워 질 수 있다.
   > 외부에는 간결한 인터페이스를 제공하고, 내부적으로는 복잡한 생성로직을 처리한다.   

## 예제코드
객체 생성을 위해 필요한 정보를 추상화된 인터페이스를 통하여 입력받고, 객체 생성 시점에 조립한다는 생각으로 설계한다.

**간단한 빌더패턴**
```java
class UrlRole {

  private final String url;
  private final Set<String> role;
  private final Set<Method> method;

  private UrlRole(String url, Set<String> role, Set<Method> method) {
    this.url = url;
    this.role = role;
    this.method = method;
  }

  static UrlRoleBuilder url(String url, Method... methods) {
    return new UrlRoleBuilder(url, methods);
  }

  @Override
  public String toString() {
    return "UrlRole{" +
        "url='" + url + '\'' +
        ", role=" + role +
        ", method=" + method +
        '}';
  }

  @SuppressWarnings({"UseBulkOperation", "ManualArrayToCollectionCopy"})
  public static class UrlRoleBuilder {
    private String url;
    private Set<String> roles;
    private Set<Method> methods;

    UrlRoleBuilder(String url, Method... methods) {
      this.url = url;
      this.roles = new HashSet<>();
      this.methods = new HashSet<>();
      for (Method each : methods) {
        this.methods.add(each);
      }
    }

    public UrlRoleBuilder hasAnyRole(String... role) {
      for (String each : role) {
        roles.add(each);
      }
      return this;
    }

    public UrlRole build() {
      validate();
      if (methods.isEmpty()) {
        for (Method each : Method.values()) {
          methods.add(each);
        }
      }
      return new UrlRole(url, roles, methods);
    }

    private void validate() {
      // do something for validate)
    }
  }
}
```

## 참고자료 
> *GoF의 디자인 패턴 p144 - p155*
