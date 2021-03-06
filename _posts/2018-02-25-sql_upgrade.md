---
layout: posts
title:  "SQL 튜닝"
date:   2018-02-25 23:50:21 +0900
comments: true
categories:
- database
tags:
- oracle
published : true
redirect_from:
- /oracle/2018/02/25/sql_upgrade/
- /oracle/sql_upgrade
---

### 서론
_DBMS는 오라클 10g를 사용했습니다._

다음 진행하게될 A프로젝트와 현재 종료직전인 B프로젝트의 유사한점 많다는 이유로 B프로젝트에 투입되어 관리자시스템 수정 및 개선 작업을 하게 되었다.
그중 기억에 남고 재밌었던 작업을 꼽으라면 SQL 튜닝이였다.
학생때부터 디비에 관심이 있어서 DB자격증 시험을 보기도 했지만 실무에서 SQL작성은 하더라도 SQL튜닝은 해본적이 없기에 골머리 앓았던 만큼 유익한 시간이였다. (_그래도 나름 쿼리속도를 생각하면서 개발을 진행했었음.._)

### 기존쿼리 분석

관리자시스템중 API연계로그를 조회하는 기능이있는데 약 400만건의 데이터가 쌓여있었고 현재 쿼리로는 조회시간이 최소 30초이상은 걸리는 치명적인 문제가 있었기에 SQL튜닝이 필수인 상황이였다.
 _추가적으로 로그테이블을 최근2주 테이블과 히스토리테이블로 분리할 계획도 있었음_

- 기존 쿼리
``` SQL
SELECT *
  FROM (
    SELECT A.*, ROWNUM AS RN
      FROM (
        SELECT API_ID,
               API_NM,
               TO_CHAR(RGST_DT, 'YYYY-MM-DD') AS RGST_DT,
               SUCCESS_YN,
               ERR_MSG
          FROM API_LOG
         ORDER BY RGST_DT DESC
      ) A
  )B
 WHERE RN BETWEEN 11 AND 20;
```
언뜻보면 일반적인 페이징 쿼리처럼 보이지만 아주 문제가 많다.
기본적으로 데이터를 페이징하여 가져오는 이유는 조회속도를 상승시키는 데에 있을것이다. ex) _전체 데이터중 필요한 데이터만 조회하고 나머지 데이터는 스캔하지 않는다._
위 쿼리는 **데이터를 전부 조회한 후 전체범위를 정렬하여**(_여기서 엄청난 비용이 발생_ ) 11번째부터 20번째, 10건의 데이터만 가져오고 있다.
**한마디로 페이징의 기능을 상실했다.**

### 결과
이런저런 삽질을 해보니 DBMS특성상 똑같은 목적을 가지고 SQL을 작성하더라도 여러방법이 있었다.
1. TOP-N 알고리즘 적용
2. 인덱스 사용

##### TOP-N 알고리즘
데이터의 조회 범위를 축소시켜 검색속도를 증가시켜주는 알고리즘
쉽게말하면 상위 N건의 데이터를 가져오는 방법이다.


예를 들면
``` SQL
SELECT *
  FROM A
 WHERE ROWNUM <= 5;
```
오라클에서 데이터 조회시 제공해주는 논리순번 ROWNUM을 이용해 TOP-N알고리즘을 적용할수 있다. 위 쿼리는 A테이블에서 5건의 데이터를 조회한후 더이상 테이블을 스캔하지 않기 대문에 A테이블에 몇건의 데이터가 있던 조회속도에는 별 차이가 없을 것이다.

그런데 ORDER BY 옵션을 적용하면 약간의 문제가 생긴다
A 테이블의 데이터가 다음과 같을때

| COL1 | COL2 |
| :------: | :-------: |
| 3 | TEST1 |
| 1 | TEST2 |
| 2 | TEST3 |
| 4 | TEST4 |


``` SQL
SELECT COL1, COL2, ROWNUM
  FROM A
 WHERE ROWNUM <= 3
 ORDER BY COL1;
```
위 쿼리의 결과는 다음과 같아진다.

| COL1    | COL2     | ROWNUM |
| :---: | :---: | :---: |
| 1       | TEST2    | 2      |
| 2       | TEST3    | 3      |
| 3       | TEST1    | 1      |


ROWNUM순서가 뒤죽박죽이 된걸 볼수있으며 페이징 쿼리작성시 문제가 생길거라는걸 알수 있을 대목이다.
그 이유는 먼저 FROM절에 의해 A테이블에 접근할 것이고 WHERE절에 명시된대로 ROW데이터가 걸러지며( __이때 ROWNUM값 부여__ ) SELECT절에 의해 뷰데이터가 생성 될 것이다.
이 모든 절차가 완료된 후 정렬이 수행되기 때문에 위 쿼리로는 원하는 ROWNUM값을 가져올수 없는것이다.

해결법은 간단하다 기존쿼리를 아래처럼 한번 감싸주면 간단하게 해결된다
``` SQL
SELECT T.*, ROWNUM AS RN
  FROM (
    SELECT COL1, COL2
      FROM A
     ORDER BY COL1
  ) T
 WHERE ROWNUM <= 3;
```
위 쿼리의 조회결과는 다음과 같다

| COL1    | COL2     | ROWNUM |
| :------: | :-------: | :-----: |
| 1       | TEST2    | 1      |
| 2       | TEST3    | 2      |
| 3       | TEST1    | 3      |


언뜻 보면 A테이블을 풀스캔한 후 COL1을 기준으로 오름차순 정렬하여 3건의 데이터만 가져와라. 이렇게 볼수 있지만 위 쿼리는 TOP-N알고리즘이 적용되어  COL1의 정렬범위가 상위3건으로 축소되어 최저값부터 3건만 정렬후 테이블스캔이 종료되어 수행비용이 절감되는 효과를 누릴수 있다.

- 실무적용 (**_잘못된 SQL_**)
``` SQL
SELECT *
  FROM (
    SELECT A.*, ROWNUM AS RN
      FROM (
        SELECT API_ID,
               API_NM,
               TO_CHAR(RGST_DT, 'YYYY-MM-DD') AS RGST_DT,
               SUCCESS_YN,
               ERR_MSG
          FROM API_LOG
         ORDER BY RGST_DT DESC
      ) A
     WHERE ROWNUM <= 20
  )B
 WHERE RN >= 11;
```
A뷰를 보면 RGST_DT를 기준으로 역순정렬하라고 명시한걸 볼수 있다. 하지만 앞서 설명한것처럼 ORDER BY 절에 의한 정렬은 데이터조회가 완료된후 즉, 데이터뷰를 기준으로 정렬 되기때문에 옵션값에 얼라이어스값 적용이 가능해진다. 때문에 `TO_CHAR(RGST_DT, 'YYYY-MM-DD') AS RGST_DT` 여기서 컬럼 얼라이어스값을 RGST_DT가아닌 다른것으로 바꿔주면 TOP-N 알고리즘이 적용되어 테이블스캔 비용시 절감되기때문에 조회시간이 훨씬 빨라진 것을 알 수있다.
