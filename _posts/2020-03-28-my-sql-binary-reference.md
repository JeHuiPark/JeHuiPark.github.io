---
layout: posts
title:  "JPA 에서 UUID 사용할 때 주의할 점"
date:   2020-03-28 22:00:00 +0900
comments: true
categories: java
tags: 
  - mysql
  - uuid
---

* TOC
{:toc}

## UUID로 조회가 안된다.

최근에 엔티티를 만들면서 `Id` 컬럼의 타입을 UUID 로 지정하면서 경험했던 이슈가 있다.

평소처럼 엔티티를 클래스를 작성하고, 기본적인 비즈니스 로직(1번 로직)을 구현하고, 비즈니스 로직에 대응하는 TC 를 작성하고 문제없이 동작하는 것을 확인한 후에 나는 다음 비즈니스 로직(2번 로직)을 구현하였다.

그런데, 다음 비즈니스 로직을 구현하고 검증하는 과정에서 문제가 발생했다.

2번 로직은 요약하면 이런 구성이였다.

1. UUID 값을 전달받는다.
    1. UUID 값이 없다면 `Exception` 을 `throw` 한다.
2. UUID 값을 이용해 조회후 값을 변경한다.

검증하는 과정에서는 정상적인 컨텍스트에서 실행을 했기 때문에 이 메소드는 예외를 발생시키지 않고 정상 리턴되야 했는데 이상하게 UUID 값이 없다는 예외를 발생시키고 있엇다.

### 혼란이 온다.

원인 파악을 위해 2번째 로직을 작성하기 전에 1번째 로직과 1번째 로직을 검증하는 TC 를 찬찬히 살펴보았다.  
이상하게도 1번째 로직과 TC 에는 아무런 문제가 없었고, 혹시나 하는 마음에 TC 의 검증로직을 다르게 작성해보기도 했지만, 당연하게도 테스트 검증결과에는 녹색불이 들어왔다.


다만, 1번 로직과 2번로직을 검증하는 환경에는 한 가지 차이점이 존재했다
- **1번 로직을 테스트할 때는 H2 인메모리 데이터베이스**를 사용했다
- **2번 로직을 테스할 때는 개발 데이터베이스**를 사용했다

### 성공적인 구글링
혹시나 해서 구글링을 해보니 [이런 글](https://phauer.com/2016/uuids-hibernate-mysql/)을 확인할 수 있었고, 중간에 몇 가지 문구가 눈에 들어왔다.  
"*UUID 에는 16 바이트가 필요합니다*"  
"*UUID 를 저장할 땐 `VARCHAR` 대신 `BINARY(16)` 컬럼을 이용하세요*"

그리고 개발 데이터베이스를 확인해보니 UUID 를 저장하는 컬럼이 `BINARY(255)` 으로 설정된 것을 확인할 수 있었다.

>[이 글](https://phauer.com/2016/uuids-hibernate-mysql/)에서 전하고자 하는 내용의 주제가 내가 필요로 하는 주제는 아니였지만, 문구 하나가 나에게는 많은 도움이 되었다.

일단 테이블 컬럼을 직접 수정하는 방법도 있지만, 개발 단계이기 때문에(?) 그냥 테이블을 삭제시켜 버리고 `Entity` 클래스의 메타정보를 수정하기로 결정하였다. (그게 더 깔끔하다고 판단)

**수정전**  
``` java
@Entity
class Example {
  @Id
  @GeneratedValue(generator = "uuid2")
  @GenericGenerator(name = "uuid2", strategy = "uuid2")
  private UUID id;

  // ...
}
```

**수정후**
``` java
@Entity
class Example {
  @Id
  @GeneratedValue(generator = "uuid2")
  @GenericGenerator(name = "uuid2", strategy = "uuid2")
  @Column(columnDefinition = "BINARY(16)")
  private UUID id;

  // ...
}
```

수정후 2번 로직을 테스트해보니 예상하던 결과가 떨어지는 것을 확인할 수 있었다.

1번 로직을 구현하고 테스트하는 시점에 이 문제를 알았다면, 더 좋았을텐데 라는 생각이 든다. 나는 이 문제를 `binary` 타입을 처리하는 DBMS의 차이(H2와 MySQL)로 이런 결과가 생긴것이라고 **추측**하고 있다.

## 왜 그럴까?
해결은 했지만, 이렇게 두리뭉술하게 넘어가기엔 너무 찝찝하다. 그래서 자료조사를 조금 더 해보았고, [오피셜 문서](https://dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html)를 발견하였다.

![image](https://user-images.githubusercontent.com/25237661/77825064-49544700-714a-11ea-98bb-6298edda3a10.png)

### 한줄 요약
저장할 때 남는 길이는 `RPAD` 처리하고 저장한다. (아 이래서 조회를 못한거구나?)

### 확인 해보자
1. 검증을 위해 (잘못된?) 테이블을 생성하자
``` sql
create table temp (
  id BINARY(255) PRIMARY KEY
)
```

1. 테스트용 데이터를 만들자.
``` sql
select uuid();
```
`8f886d50-70ff-11ea-b498-02dd0a2dce82` 라는 `UUID` 를 생성했다.  

1. UUID의 길이를 확인해보자.  
UUID의 길이는 16바이트의 길이를 요구한다고 하였다. 눈으로 직접 확인해보자
``` sql
select length(unhex(replace('8f886d50-70ff-11ea-b498-02dd0a2dce82','-','')))
```
![image](https://user-images.githubusercontent.com/25237661/77825438-acdf7400-714c-11ea-9fae-955273b19a67.png)  
<br>
약속대로 16 이라는 길이가 나온다.

1. 데이터를 저장해보자  
`MySQL` 에서 가이드 하는대로 `-` 를 공백으로 치환하고 `binary` 타입으로 변경하여 데이터를 밀어 넣자.  
```sql
insert into temp 
values (unhex(replace('8f886d50-70ff-11ea-b498-02dd0a2dce82','-','')));
```

1. 조회를 해보자  
    레퍼런스에서는 남는 공간은 패딩처리 한다고 했다.  
    그러니까 나는 16바이트의 길이를 가진 데이터를 저장했지만, 실제로는 255 길이를 소유한 데이터가 있어야 한다.  
    <br>
    그걸 눈으로 확인하기 위해 아래와 같은 쿼리를 날렸다.
    ``` sql
    select length(id), hex(id)
      from temp;
    ```
    ![image](https://user-images.githubusercontent.com/25237661/77825566-6fc7b180-714d-11ea-8b53-c417115a261a.png)  
    <br>

    레퍼런스에서 설명한대로 우측에 패딩값이 들어간 것을 확인할 수 있다.  
    미리 생성해둔 UUID 로 조회를 시도해보자

    > 내가 경험한 이슈를 흉내낸 것이다  

    ``` sql
    select *
      from temp
    where id = unhex(replace('8f886d50-70ff-11ea-b498-02dd0a2dce82','-',''));
    ```
    조회가 안된다.  
    패딩값 때문에 당연한 결과일 것이다.
    <br>
    **그렇다면 조회조건에 패딩값을 포함하면 조회가 되야 되겠지?**
    ``` sql
    select hex(SUBSTR(id, 1, 16)) AS origin_uuid
      from temp
    where id = rpad(unhex(replace('8f886d50-70ff-11ea-b498-02dd0a2dce82','-','')), 255, '\0');
    ```

    예상대로 조회가 되는 것을 확인할 수 있다.  
    ![image](https://user-images.githubusercontent.com/25237661/77825850-47d94d80-714f-11ea-94f2-4068c05b2f1e.png)
