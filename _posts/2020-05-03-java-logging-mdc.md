---
layout: posts
title:  "Java 로깅전략 with MDC"
date:   2020-05-03 20:00:00 +0900
comments: true
categories: java
---

* TOC
{:toc}

어플리케이션에서 이루어지는 어떤 행위에는 보통 Context(문맥) 가 존재한다.  
그렇기에 로그정보를 Context 단위로 볼 수 있다면, 더 유의미한 로그가 될 것이다.  

## MDC
- Java 로깅 프레임워크에서는 관련 기능을 `MDC` 라는 이름으로 제공하고 있다.  
- `MDC` 는 `key/value` 저장소를 지원하며, 이 저장소는 `ThreadContext` 에 의존한다.

### 예제코드

**의존성**  
```groovy
dependencies {
    compile group: 'org.apache.logging.log4j', name: 'log4j-slf4j-impl', version: '2.13.2'
    compile group: 'org.apache.logging.log4j', name: 'log4j-api', version: '2.13.2'
    compile group: 'org.apache.logging.log4j', name: 'log4j-core', version: '2.13.2'
}
```

**로그설정**  
`%-8mdc{trace-id}` <- mdc 포멧
> 자세한 정보는 참고자료에 명시한 Log4j2 Manual 에서 확인할 수 있다.
 
```xml
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout>
                <pattern>
                    {% raw %}%highlight{%d{HH:mm:ss.SSS} [%-5t] [%-8mdc{trace-id}] [%-5level] %msg%n%throwable}{% endraw %}
                </pattern>
            </PatternLayout>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="debug">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

**Java 코드**    
```java
class MockApplication {
  final ExecutorService executorService = Executors.newFixedThreadPool(4, Executors.privilegedThreadFactory());

  void request() {
    executorService.submit(() -> {
      MDC.put("trace-id", UUID.randomUUID().toString().substring(0,8));
      // do something
      MDC.clear();
    });
  }
}
```

**실행결과**  
![image](https://user-images.githubusercontent.com/25237661/80915551-f5163580-8d8d-11ea-94c0-f71259c4080d.png)  

[예제코드 저장소](https://github.com/JeHuiPark/java-sample/tree/master/logger-example)  

## 참고자료
[log4j2 manual](http://logging.apache.org/log4j/2.x/manual/configuration.html)