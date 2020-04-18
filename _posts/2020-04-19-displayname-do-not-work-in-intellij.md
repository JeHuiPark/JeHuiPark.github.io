---
layout: posts
title:  "JUnit5 DisplayName 이 동작하지 않을 때"
date:   2020-04-19 01:00:00 +0900
comments: true
categories: note
---

* TOC
{:toc}

`@DisplayName` of junit5 in intellij do not working

## `@DisplayName` 이 동작하지 않아요

JUnit5 를 이용하여 아래와 같은 테스트를 작성하고 
``` java
@DisplayName("example")
class JunitExample {

  @DisplayName("when coding enjoy")
  @Nested
  class Context_Sample {

    @Test
    @DisplayName("i am happy")
    void test1() {
    }
  }
}
```
이런 결과를 기대했지만?  
![image](https://user-images.githubusercontent.com/25237661/79642932-3974d380-81db-11ea-885d-0d5f033561b1.png){:width="300px"}

이런 화면이 나온다면..😭  
![image](https://user-images.githubusercontent.com/25237661/79642973-780a8e00-81db-11ea-9a50-73fbc778b980.png){:width="300px"}

## 해결
`Run tests using` 옵션을 변경 `gradle` -> `intellij`  
![image](https://user-images.githubusercontent.com/25237661/79643067-0ed74a80-81dc-11ea-95bd-2a163e6b1bbf.png)

