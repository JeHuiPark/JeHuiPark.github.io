---
layout: posts
title:  Springboot Embedded Mongo DB 이슈
date:   2021-04-29 22:00:00 +0900
comments: true
categories: spring
tags:
  - java
  - mongodb
  - spring
---

* TOC 
{:toc}
  
## $lookup 동작X

간단한 구조의 서브 도큐먼트일 경우 `$lookup` aggregation 파이프라인은 예상하는 결과를 출력 하지만,  
**배열에 속하는 멀티 서브 도큐먼트**에서 `$lookup` aggregation 파이프라인은 **예상과 다른 결과를 출력**한다. 

### 실행 환경

lib | version
--- | ---
org.springframework.boot:spring-boot-starter-data-mongodb | 2.2.4
de.flapdoodle.embed:de.flapdoodle.embed.mongo | 2.2.0


### 발생조건

Collection: aa
```javascript
db.aa.insert({"_id": 1, "name": "aa-name-1"});
db.aa.insert({"_id": 2, "name": "aa-name-2"});
```

Collection: bb
```javascript
db.bb.insert({
  "_id": 1,
  "name": 'bb-name',
  "bb_subs": [
    { "sub-doc-id": "sub-document-1", "ref_id": 1 },
    { "sub-doc-id": "sub-document-2", "ref_id": 2 }
  ]
});
```

Aggregation query
```javascript
db.bb.aggregate([
    {
        "$match":{
            "_id":1
        }
    },
    {
        "$lookup":{
            "from":"a",
            "localField":"bb_subs.ref_id",
            "foreignField":"_id",
            "as":"detail"
        }
    }
])
```

실행결과
```json
{
    "_id" : 1.0,
    "name" : "bb-name",
    "bb_subs" : [ 
        {
            "sub-doc-id" : "sub-document-1",
            "ref_id" : 1.0
        }, 
        {
            "sub-doc-id" : "sub-document-2",
            "ref_id" : 2.0
        }
    ],
    "detail" : []
}
```

### 해결방법
Embedded MongoDB 버전을 특정 버전 이상으로 변경한다. (> 3.5.8)

```yml
spring:
  mongodb:
    embedded:
      version: 3.6.5
```

> [Springboot 2.2.4 기준으로 기본 버전][스프링부트 기본버전] 은 `3.5.5` 버전  
Embedded MongoDB 라이브러리에서 지원하는 버전은 [`de.flapdoodle.embed.mongo.distribution.Version`][몽고디비 지원버전]
에서 확인할 수 있다

### 원인
MongoDB Bug (https://jira.mongodb.org/browse/SERVER-28717)   
Fixed Version`3.4.6`, `3.5.8`

[스프링부트 기본버전]: https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mongo/embedded/EmbeddedMongoProperties.java
[몽고디비 지원버전]: https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo/blob/master/src/main/java/de/flapdoodle/embed/mongo/distribution/Version.java
