---
layout: post
title:  "마이바티스 옵션"
date:   2018-01-31 17:59:21 +0900
comments: true
categories:
  - mybatis
tags:
  - mybatis
  - spring
---

- ### insert, update, delete 요청에서 selectKey추출

```xml
<insert id="id" parameterType="HashMap" >
  <selectKey keyProperty="seq" resultType="Integer" order="BEFORE">
    (SELECT SEQUNCE_NAME.NEXTVAL FROM DUAL)
  </selectKey>
  INSERT INTO TABLE_NAME
  VALUES(#{seq}, #{data})
</insert>
```
##### serviceImpl.java
```java
reqMap.get("seq") // null
dao.insert(id, reqMap);
reqMap.get("seq") // SEQUNCE_NAME.NEXTVAL  
```
<br><br>

- ### 반복문  

```xml
<delete id="id" parameterType="HashMap">
DELETE TABLE_NAME WHERE COL = #{key}
<if test="cttNums != null ">
    <foreach collection="cttNums"  item="item" separator="," open="AND CTT_NUM NOT IN (" close=")">#{item}</foreach>
 </if>
</delete>
```

----
<br>

### 내가 mybatis에서 경험했던 이슈

#### 1. sqlmap에서 if옵션을 사용해 분기처리를 하는부분이 있는데 의도한대로 분기처리가 되지 않았다.

> ##### 문제의 sqlmap
```xml
<select id="...." parameterType="HashMap" resultType="egovMap">
    SELECT
....
  FROM ....
WHERE .... = #{....}
AND DEL_FLAGE = 'N'
    <if test="fileTy == '5'">
    ...sql
</if>
    <if test="fileTy == '6'">
          AND LOAN_FILE_TYPE = #{fileTy}
    </if>
ORDER BY ....
</select>
```

> ###### 해결 과정
> 1. **파라미터 확인**
>     ```java
>     reqMap.get("fileTyp") // "5"
>     ```
>    <br>
> 2. **파라미터 타입 확인**
>     ```java
>     reqMap.get("fileTy") instanceof String  // TRUE
>     reqMap.get("fileTy") instanceof Integer // FALSE
>     ```
>    <br>
> 3. **혹시 mybatis에서 파라미터값에 따라 자료형을 재정의하지 않을까 ??**
>     1. sqlmap 소스변경 기존 `<if test="fileTy == '5'">` 이부분에서 싱글쿼테이션을 제거하여 정수형으로 변경 `<if test="fileTy == 5 ">`
>
>
>~~테스트 완료~~
>**정정합니다.** <span style="margin-right:20px;"></span>_(2018.09.01)_
> **mybatis에서 파라미터값에 따라 자료형을 재정의하진 않습니다.**<br>
> 위와같은 상황에서 **mybatis에 개발자가 원하는걸 명확히 전달하려면** 아래와 같이 사용하면 됩니다.

```xml
<!-- test구문을 싱글쿼테이션으로 감싸주는게 중요. -->
<if test='param.equals("5")'>
  ...
</if>
```
>왜 그런지에 대한 이유는 [다른글](https://jehuipark.github.io/mybatis/2018/09/01/mybatis_ognl/)에서 별도로 다루겠습니다.
[mybatis의 OGNL기반 표현식 분석으로 이동](https://jehuipark.github.io/mybatis/2018/09/01/mybatis_ognl/)
