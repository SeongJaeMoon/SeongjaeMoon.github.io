---
layout: post
title:  "Oracle 뷰"
date:   2018-02-18 23:00:00
author: Seongjae Moon
categories: DB
tags:   Oracle 오라클 뷰 쿼리
---

끝나지 않을 것만 같던 설 연휴가 끝나버렸다. 설 연휴의 끝은 역시 블로그 포스팅! [저번 포스팅](https://seongjaemoon.github.io/2018/02/03/database-oracle4/)에 이어서 오라클 뷰(VIEW)에 대해 정리를 해보자.
### 먼저, 뷰에 대해서 알아보자.
뷰란 이미 특정한 데이터베이스 내에 존재하는 하나 이상의 테이블에서 사용자가 얻기 원하는 데이터들만을 정확하고 편하게 가져오기 위하여 사전에 원하는 컬럼들 만을 모아서 만들어 놓은 가상의 테이블로, **편리성 및 보안에 목적**이 있다.

가상의 테이블이란 뷰가 실제로 존재하는 테이블이 아니라 하나 이상의 테이블에서 파생된 또 다른 정보를 볼 수 있는 방법이며 그 정보들을 추출해내는 SQL 문장이라고 볼 수 있다.
#### 뷰의 생성 방법을 알아보자.
```sql
-- 일반적으로 뷰를 생성할 땐, 기본 테이블이 존재한다고 가정하고 만들기 때문에 FORCE 옵션을 사용하면, 기본 테이블의 존재 여부에 상관없이 뷰를 생성한다.
-- 기존의 VIEW의 구조를 변경하여 덮어쓰고자 할 땐 OR REPLACE 옵션을 붙이면 된다.
 CREATE [OR REPLACE] [FORCE | NOFORCE] VIEW 뷰이름
 [(alias[, alias]...]
 AS subquery
 [WITH CHECK OPTION] -- 해당 뷰를 통해서 조건 컬럼 값을 변경하지 못하게 하는 옵션이다. 예를 들어, 직위가 과장인 뷰를 생성했다면, 직위를 대리로 변경 불가능하다.
 [WITH READ ONLY]; -- 뷰를 이용한 쿼리문이 검색 쿼리(SELECT)로 한정된다.
```
#### 뷰의 삭제 방법을 알아보자.
```sql
-- 뷰 생성 권한을 부여 받은 경우만 실행 가능    
DROP VIEW 뷰이름;
```
#### 인라인 뷰
인라인 뷰는 FROM 절에서 서브 쿼리를 사용하여 생성한 임시 뷰이다. 인라인 뷰는 SQL 문이 실행되는 동안만 임시적으로 정의된다. 즉, 객체로서 저장되지 않는다.
(뷰 생성 권한 없어도 실행 가능.)
```sql
-- 특정 순위까지만 출력하는 쿼리를 작성한다고 가정하면
SELECT *
    FROM (SELECT ROW_NUMBER() OVER(ORDER BY name_) AS num_
    , eid, name_, phone
    FROM employees)
    WHERE num_ <= 5;
```
#### WITH CHECK OPTION 지정 뷰
WITH CHECK OPTION 절을 사용하면 뷰를 통해 참조 무결성(reference integrity)을 검사할 수 있고 DB 레벨에서의 constraint 적용이 가능하다.
```sql
-- 예를 들어, 사원 정보를 보여주는 뷰를 만들 때를 가정해보자.
--' 사원' 직위를 가진 직원 정보를 보여주는 뷰를 생성한다.
CREATE OR REPLACE VIEW emp_jobs_view
AS
SELECT *
FROM (SELECT emp_id, name_ , ssn, hiredate, phone
, e_.job_id, basicpay, extrapay
FROM emp e_, jobs j
WHERE e_.job_id = j.job_id)
WHERE job_id =  (SELECT job_id FROM jobs WHERE job_title='사원');

-- 뷰를 이용한 자료 입력 (직위가 '사원'인 아닌 경우 준비한다.)
INSERT INTO emp_jobs_view (emp_id, name_ , ssn, hiredate, phone
, job_id, basicpay, extrapay)
VALUES ('1200', '홍길동', '901212-1234567'
, '2010-10-05', '010-1212-3434'
, (SELECT job_id FROM jobs WHERE job_title='과장')
, 2000000, 1000000);

SELECT * FROM emp_jobs_view;
--> 신규 자료 검색 불가.
--> '사원'이 아닌 자료는 검색되지 않는다.
--> 뷰를 이용한 입력시 잘못된 자료가 입력되는 것을 막을 수 없다.
--> WITH CHECK OPTION 지정 필요.

SELECT * FROM emp;
--> 원본 테이블에서는 검색 가능.
```

예전에 대학에서 교수님께서 뷰를 창문으로 비유했던 것이 생각난다. ~~갑자기?~~ 창문 너머로 볼 수 있는 내부는 한정적이고(보안), 특정 부분만 보게(편리성) 할 수 있기 때문에 창문이라고 표현 하셨던 것 같다. 물론, 그땐 저게 뭔 소리람 했지만.

교수님의 비유에 새삼 감탄하며? 다음은 이러한 뷰를 이용해 자바 코드 상에서 편리하게 활용하는 방법과 검색 쿼리에 대해 더 알아보는 걸로~

* 오타나 잘못된 부분을 지적 해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* 참고문헌<br>
서진수, 김도균 지음 다양한 예제로 쉽게 배우는 오라클 SQL과 PL/SQL<br>
미크 지음, 윤인성 옮김 DB 성능 최적화를 위한 SQL 실전 가이드 SQL 레벨업
