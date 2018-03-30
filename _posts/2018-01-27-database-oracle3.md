---
layout: post
title:  "Oracle 조인 쿼리 및 스키마 정리"
date:   2018-01-14 23:00:00
author: Seongjae Moon
categories: Database
tags:   Oracle 오라클 조인 스키마 쿼리
---

[지난 포스팅](https://seongjaemoon.github.io/database/2018/01/14/database-oracle2.html)에 이어 조인과 스키마에 대해 간단하게 알아보자. 우선 조인은 결합 구문으로,  DB 내의 여러 테이블의 공통 값(PK-FK 관계)을 통해 둘 이상의 테이블에서 데이터를 검색할 수 있게 해준다.

조인은 서브 쿼리와 같은 결과를 반환할 수 있으며, 조인으로 해결 가능한 경우라면 성능면에서 조인이 더 낫다고 한다.

#### 조인 하는 방법에 따라 종류가 나눠진다.
* INNER JOIN - 공통 값을 가지는 조건을 만족하는 ROW만 출력
* OUTER JOIN - 공통 값을 가지는 조건을 만족하는 경우 외에도 출력
* EQUI JOIN - =(equal) 연산자를 이용한 조건을 사용한 JOIN
* NON-EQUI JOIN - =(equal) 연산자 외의 연산자를 이용한 조건을 사용한 JOIN
* SELF JOIN - 자기 자신을 두 개의 가상 테이블로 만들어서 JOIN하는 방법. (별칭 사용.)
* ANSI JOIN - JOIN 구문을 오라클 방법이 아니라 표준적인 방법으로 작성한 것.

#### EQUI JOIN
두 개 이상의 테이블에 관계되는 컬럼들의 값들이 일치하는 경우에 사용하는 가장 일반적인 join 형태로 **WHERE 절에 '='(등호)를 사용한다. 흔히 PRIMARY KEY, FOREIGN KEY 관계를 이용**하며, 오라클에서는 NATURAL JOIN이 EQUI JOIN과 동일하다. 또는, USING 절을 사용하여 EQUI JOIN과 동일한 결과를 출력 한다.
**PRIMARY KEY, FOREIGN KEY 제약이 걸려있는 컬럼은 값들이 일치하도록 되어 있다.**
```sql
SELECT *
FROM 테이블_명1, 테이블_명2
WHERE 테이블_명1.컬럼_명1=테이블_명2.컬럼_명1;

--ANSI SQL
 SELECT 테이블_명1.컬럼_명, 테이블_명2.컬럼_명
 FROM 테이블_명1 JOIN 테이블_명2
 ON 테이블_명1.컬럼_명1=테이블_명2.컬럼_명1
 WHERE 조건식
```
#### Non-EQUI JOIN
Join 조건에서 = 연산이 아닌 모든 경우를 말한다.
비교 대상인 컬럼 간에 연관성이 없어도 가능하다.
```sql
SELECT 컬럼리스트(별칭 사용 필요)
FROM 테이블1 별칭1, 테이블2 별칭2
WHERE 별칭1.컬럼 <= 별칭2.컬럼;

SELECT 컬럼리스트(별칭 사용 필요)
FROM 테이블1 별칭1, 테이블2 별칭2
WHERE 별칭1.컬럼 >= 별칭2.컬럼;
```
#### Outer Join
양쪽 테이블에 동일한 값이 존재하는 경우만 출력되는 경우는 Equi Join(Inner Join)이고,
동일한 값이 없어도 한 쪽 테이블의 자료는 모두 출력되는 경우는 Outer Join이다.

LEFT OUTER JOIN(테이블2가 NULL이 출력되는 상태)
테이블1은 전체 출력, 테이블2는 부분 출력(Equi Join 조건 만족하는 경우만 출력) 짝이 없는 경우 NULL로 출력된다. (+)는 NULL 출력 대상쪽에 붙인다. **OUTER JOIN에 사용한 컬럼은 일반 조건식에서도 (+)를 모두 붙여야 한다.**
```sql
--RIGHT OUTER JOIN(테이블1이 NULL이 출력되는 상태)
SELECT 컬럼리스트
 FROM 테이블1 별칭1, 테이블2 별칭2
 WHERE 별칭1.컬럼(+) = 별칭2.컬럼;

--ANSI LEFT OUTER JOIN(테이블1의 자료는 모두 출력)
SELECT 컬럼리스트
 FROM 테이블1 별칭1 LEFT OUTER JOIN 테이블2 별칭2
 ON 별칭1.컬럼 = 별칭2.컬럼;
```
#### Self Join
일반적인 Join은 두 개의 서로 다른 테이블을 대상으로 Join을 하지만,
Self Join은 자기 자신을 가상의 테이블(테이블1, 테이블2가 모두 자기 자신인 상태)로 생각하고 Join을 진행하는 것. **별칭 사용 필수**
```sql
SELECT 컬럼리스트
FROM 테이블 별칭1, 테이블 별칭2
WHERE 별칭1.컬럼 = 별칭2.컬럼;

SELECT 컬럼리스트
FROM 테이블 별칭1, 테이블 별칭2
WHERE 별칭1.컬럼 <= 별칭2.컬럼;

SELECT 컬럼리스트
FROM 테이블 별칭1, 테이블 별칭2
WHERE 별칭1.컬럼 >= 별칭2.컬럼;
```
#### 테이블 3개 이상 JOIN
```sql
SELECT 컬럼리스트
FROM 테이블1, 테이블2, 테이블3, ...
WHERE 테이블1.컬럼명 = 테이블2.컬럼명
AND 테이블2.컬럼명 = 테이블3.컬럼명
AND ... ;
```

#### DB 스키마란?
DB에서 Schema(스키마)란 말 그대로 구조를 나타낸다. 관계형 데이터베이스의 3대장인 구조, 제약조건, 연산 중 하나라고 할 수 있다. 보통 **개념적 모델링 > 논리적 모델링 > 물리적 모델링**으로 이루어지며, 각 단계마다 다른 산출물을 통해 설계된다.

 > 개념적 모델링
일반적으로 E-R 다이어그램을 통해 표현하며, 명사적 속성을 나타내는 Entity(개체), 개체의 특징을 나타내는 Attribute(속성), 값의 범위를 나타내는 Domain(도메인), 동사적 속성을 나타내는 Relationship(관계)로 표현힌다. 이 단계에선, DBMS는 신경쓰지 않는다.  

 > 논리적 모델링
개념적 모델링에서 나온 산출물을 토대로 논리적 설계를 하는 단계로,  각각 Entity -> Table(테이블), Attribute -> Column(컬럼), Domain -> Constraint(제약조건) Relationship -> Foreign Key(외래키 참조 관계)로 대응되게 된다. 이 단계에서, 대망의 '정규화' 과정을 거쳐 갱신 이상, 삭제 이상, 삽입 이상 등의 이상 현상이 발생하지 않도록 테이블의 컬럼을 쪼개는? 작업을 거치게 된다.

> 물리적 모델링
실제 SQL 쿼리문으로 물리적으로 DB를 설계하는 단계로, CREATE 쿼리를 통해 테이블을 생성하는 것이 대표적인 예라고 할 수 있다.

이렇게, 조인 쿼리문과 스키마에 대해 간단하게 알아보았다.  스키마같은 경우엔 외부, 개념, 내부 스키마 등으로 나뉘어지는데, 이는 바라보는 관점에 따라 달라지게 된다. ~~오늘은 귀찮으니 이만...~~ DB는 역시 공부할게 차고 넘친다. 헿.  

[다음](https://seongjaemoon.github.io/database/2018/02/03/database-oracle4.html) 포스팅에선 제약 조건 및 DDL, DML, DCL 등에 대해 심도있게 알아보는 걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* 참고문헌<br>
서진수, 김도균 지음 다양한 예제로 쉽게 배우는 오라클 SQL과 PL/SQL<br>
미크 지음, 윤인성 옮김 DB 성능 최적화를 위한 SQL 실전 가이드 SQL 레벨업
