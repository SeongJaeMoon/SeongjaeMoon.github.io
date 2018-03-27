---
layout: post
title:  "Oracle 조인 쿼리 및 스키마 정리"
date:   2018-02-03 23:00:00
author: Seongjae Moon
categories: DB
tags:   Oracle 오라클 DDL DCL DML
---

2018년 올 해도 벌써 1월이 다 가버렸다. 이럴수가.. 2월의 첫 주말을 맞아 오늘은 그 동안 못 했던 개인적인 공부와 블로그 포스팅을 열심히 해봐야겠다. [저번 포스팅](https://seongjaemoon.github.io/2018/01/27/database-oracle3/)에 이어서 오늘은 DB 제약 조건 및 DDL, DML, DCL에 대해 알아보자.

우선, 데이터베이스에 저장되는 것들은 테이블 외에 기타 여러 가지 것들이 저장되는데 이것을 데이터베이스 객체(Database Object)라고 부른다. 테이블의 제약 조건 등이 함께 저장 되는 것을 생각하면 되겠다. 그렇다면, DDL에 대해 알아보자.

 DDL(Database Definition Language)는 말 그대로 데이터베이스 정의 구문으로, 테이블을 새로 구성하거나, 변경 및 삭제 하는 작업을 가능케 하는 쿼리문이라고 할 수 있다.

### DDL 구문에 사용될 수 있는 용어에 대해 알아보자.

- TABLE
하나 또는 여러 컬럼(Column)들이 모여 하나의 레코드를 이루며, 이러한 레코드들이 모여 테이블이 된다. 예를 들어 "사원" 테이블은 사번, 이름, 부서 등 여러 컬럼을 갖게 되고, 사원 수 만큼의 레코드를 갖게 된다.
- CREATE 문
데이터베이스 내의 모든 객체를 생성할 때 사용하는 문장
- ALTER 문
이미 생성된 객체의 구조를 변경
- DROP 문
생성된 객체를 삭제
=>주의) 관리자에 의해 RESOURCE 권한을 부여 받은 사용자만 작업 가능

#### CREATE 쿼리문에 대해 알아보자.
테이블은 관계 데이터베이스에 데이터 저장을 위해 이용되는 객체이며, 행과 열 을 통해 spread sheet와 비슷한 방식으로 데이터를 표시한다.
예를 들어, 성적 정보 저장용 테이블이 있다면, 아래와 같이 저장된다고 생각하면 된다.

column|column|column|column|column
:--:|:--:|:--:|:--:|:--:|:--:
번호(PK)|이름|국어|영어| 수학|
1| 홍길동|100|100 |100|  -> row
2|박길동| 90|80| 90|  -> row
3|최길동|100 |70 | 80 | -> row

```sql
--SQL 쿼리문은 아래와 같은 형식으로 작성하게 된다.
CREATE [GLOBAL TEMPORARY] TABLE [스키마.]테이블_이름 (
    열_이름 데이터타입 [DEFAULT 표현식] [제약조건]
    [, 열_이름 데이터타입 [DEFAULT 표현식] [제약조건] ]
    [ ,...]
);
```
테이블 생성시 PK 제약을 지정할 수 있는 컬럼을 반드시 추가할 것.
예를 들어, 성적 정보 저장용 테이블을 만든다면,
```sql
--테이블 생성
CREATE TABLE sungjuk (
  sid NUMBER --고유번호 저장용 컬럼(PK). 필수 항목.
  ,name VARCHAR2(20) --이름 저장용 컬럼. 영숫자 20자 이내. 한글 저장시 NVARCHAR2(20) 한글 20자 이내.
  ,kor NUMBER(3) --과목 점수 저장용 컬럼. 0~100
  ,eng NUMBER(3) --과목 점수 저장용 컬럼. 0~100
  ,mat NUMBER(3) --과목 점수 저장용 컬럼. 0~100
);
--테이블 존재 확인
SELECT * FROM user_tables;
```
#### 데이터 타입(자료형)
오라클이 제공하는 데이터 타입은 **단일 값을 저장하는 스칼라 데이터 타입**, **여러 개의 데이터를 저장할 수 있는 컬렉션 데이터 타입** 그리고, **컬럼이 다른 테이블 객체를 참조하는 관계 데이터 타입**이 있다. CREATE와 더불어 데이터 타입도 함께 알아보자.
##### VARCHAR2
- 형식 : VARCHAR2(n)
- 가변 길이 문자 데이터를 저장하며 최대 길이는 4000자이고, 반드시 길이를 명시해야 한다. NLS(국가별 언어 집합)는 한글과 영문만 가능하다.
- VARCHAR는 최대 2000개 문자를 저장하며 VARCHAR2와는 다르게 VARCHAR(10)로 선언하면 null을 채워 실제로는 10개의 공간을 사용한다.
- VARCHAR2(10)는 필요한 문자까지만 저장하는 variable length이며 최대 4000개 문자까지 저장할 수 있다. **(한글 저장용 NVARCHAR2(n) 자료형)**
##### NUMBER
- 형식 : NUMBER(P, S)
- **(정수나 실수 저장하기 위한 가변길이의 표준 내부 형식이다.)**
- P(1~38)는 정밀도로 전체 자리수를 나타내며 기본 값이 38이다.
- S(-84~127)는 소수점 이하의 자릿수이다.
##### DATE
- 『년/월/일 시:분:초』까지 저장하며, 기본적으로 년/월/일 정보를 출력한다.

#### 제약 조건에 대해 알아보자.
제약 조건에 대해 살펴보기 전에, 무결성이란 것에 대해 알아보자. 무결성에는 크게 개체 무결성(Entity Integrity), 참조 무결성(Relational Integrity), 도메인 무결성(Domain Integrity)이(가) 있다.
- 개체 무결성
개체 무결성은 릴레이션에 저장되는 튜플(tuple)의 유일성을 보장하기 위한 제약조건이다.
- 참조 무결성
참조 무결성은 릴레이션 간의 데이터의 일관성을 보장하기 위한 제약조건이다.
- 도메인 무결성
도메인 무결성은 속성에서 허용 가능한 값의 범위를 지정하기 위한 제약조건이다.
예를 들어, 학생 정보 저장용 테이블을 만든다면 아래와 같이 생성한다.
```sql
--테이블 생성
CREATE TABLE member (     --테이블 이름 member
    sid NUMBER        --고유번호 저장용 컬럼
    ,name VARCHAR2(10) --이름, 10글자만 허용
    ,kor NUMBER(3)     --국어, 숫자 3자리만 허용(0~999). 0~100 제한.
    ,eng NUMBER(3)     --영어, 숫자 3자리만 허용
    ,mat NUMBER(3)     --수학, 숫자 3자리만 허용
);
```

#### 제약조건 종류
- PRIMARY KEY(PK) : 해당 컬럼 값은 반드시 존재해야 하며, 유일해야 함(NOT NULL과 UNIQUE 제약조건을 결합한 형태)
- FOREIGN KEY(FK) : 해당 컬럼 값은 참조되는 테이블의 컬럼 값 중의 하나와 일치하거나 NULL을 가짐
- UNIQUE KEY(UK) : 테이블내에서 해당 컬럼 값은 항상 유일해야 함
- NOT NULL : 컬럼은 NULL 값을 포함할 수 없다.
- CHECK(CK) : 해당 컬럼에 저장 가능한 데이터 값의 범위나 조건 지정

##### PRIMARY KEY (PK)
- 테이블에 대한 기본 키를 생성한다.
- 테이블에서 각 행을 유일하게 식별하는 컬럼 또는 컬럼의 집합이다. 기본 키는 테이블 당 하나만 존재한다. 그러나, 반드시 하나의 컬럼으로 만 구성되는 것은 아니다. **NULL 값이 입력될 수 없고, 이미 테이블에 존재하고 있는 데이터를 다시 입력할 수 없다.**
(UNIQUE INDEX가 자동으로 만들어 진다.)
```sql
-- PK 지정 테스트 -  (테이블 생성시 제약 추가) 테이블 레벨의 형식
--> 제약명을 사용자가 결정한다
--> 제약명은 '테이블명_컬럼명_PK' 형식으로 작성한다.
CREATE TABLE test (
    col1 NUMBER(3)
    ,col2 VARCHAR2(10)
    ,CONSTRAINT TEST2_COL1_PK PRIMARY KEY(col1)
);
-- PK 지정 테스트 - 다중 컬럼 PK 지정 (복합키)
CREATE TABLE test (
    col1 NUMBER(3)
    ,col2 VARCHAR2(10)
    ,CONSTRAINT TEST3_COL1_COL2_PK PRIMARY KEY(col1, col2) -- 두 개가 묶여서 하나의 Primary Key(복합키), 두 개가 동시에 같을 때만 중복으로 고려.
);
ALTER TABLE test
ADD CONSTRAINT TEST_COL1_PK PRIMARY KEY(col1);
-- constraint를 추가 시에 constraint 명을 생략하면 오라클 서버가 자동적으로 constraint 명을 부여한다.
-- 일반적으로 constraint명은 테이블명_컬럼명_constraint약자'처럼 기술한다.

--(추가)기존 테이블 구조를 복사해서 새 테이블 생성 (제약조건은 복사되지 않는다)
CREATE TABLE 새테이블명
AS
SELECT 컬럼리스트 FROM 기존테이블명 WHERE 1=0;
```

##### FOREIGN KEY (FK)
- 참조 키 또는 외래 키(FK)는 **두 테이블의 데이터 간 연결을 설정하고 강제 적용하는 데 사용되는 열**이다. 한 테이블의 기본 키 값이 있는 열을 다른 테이블에 추가하면 테이블 간 연결을 설정할 수 있다. 이 때 두 번째 테이블에 추가되는 열이 외래 키가 된다.  (외래 키가 적용된 컬럼에는 데이터 입력시 기본 키 값 범위 내에서만 입력 가능.)
- 부모 테이블이 먼저 생성된 후, 자식 테이블(foreign key를 포함하는 테이블)이 생성되어야 한다.

###### FOREIGN KEY 생성 시 주의사항
1. 참조하고자 하는 부모 테이블을 먼저 생성해야 한다.
2. 참조하고자 하는 컬럼이 PRIMARY KEY 또는 UNIQUE 제약조건이 있어야 한다.
3. 테이블 사이에 PRIMARY KEY와 FOREIGN KEY가 정의 되어 있으면, primary key 삭제 시 foreign key 컬럼에 그 값이 입력되어 있으면 삭제가 안 된다. (단, FK 선언 때 ON DELETE CASCADE나 ON DELETE SET NULL옵션을 사용한 경우에는 삭제된다.)
4. 부모 테이블을 삭제하기 위해서는 자식 테이블을 먼저 삭제해야 한다.
5. 서로 제약에 의해 참조하는 경우는 서로 삭제되지 않는다. 제약조건을 먼저 삭제해야 한다.
```sql
테이블 생성후 제약 추가
ALTER TABLE 테이블명
    ADD CONSTRAINT 제약명 FOREIGN KEY(컬럼명)
            REFERENCES 참조_테이블명 (참조_컬럼명);
            [ON DELETE CASCADE | ON DELETE SET NULL]
            1. 기존 테이블 구조(컬럼명, 자료형) 확인
```

##### UNIQUE
-PK와 유사하게 유일성을 보장하는 제약
-NULL 허용, 하나의 테이블에 여러번 지정 가능.
```sql
ALTER TABLE 테이블명
 ADD CONSTRAINT 테이블명_컬럼명_UK UNIQUE(컬럼명);
```
##### CHECK
-특정 조건을 만족하는 경우만 입력 허용하는 제약. 예를 들어, 정수 입력시 양수만 입력 가능.
-하나의 테이블에 여러번 지정 가능.
```sql
ALTER TABLE 테이블명
 ADD CONSTRAINT 테이블명_컬럼명_CK CHECK(조건식);
```
#### ALTER와 DROP 쿼리문에 대해 알아보자.

1. 기존 테이블에 새 열 추가
```sql
ALTER TABLE 기존테이블명
    ADD (열이름 자료형, ...);
--> 기존 테이블에 데이터가 있는 경우는 새로 만들어진 컬럼의 데이터는 NULL만 채워진다.
--> NULL이 채워진 컬럼에 자료를 채우려면 UPDATE 명령을 이용한다.
--> 컬럼 생성시 자동으로 기본값을 채우려면 DEFAULT 키워드를 이용한다.
```
2. 기존 테이블에서 기존 열 자료형 변경
```sql
ALTER TABLE 기존테이블명
    MODIFY (기존열이름 새로운자료형);
--> 기존 테이블에 데이터가 있는 경우는 새로운 자료형이 기존 자료에 적합해야 한다.
```
3. 기존 테이블에서 기존 열 이름 변경
```sql
ALTER TABLE 기존테이블명
    RENAME COLUMN 기존열이름 TO 새열이름;
--> 기존 테이블에 데이터가 있어도 가능하다.
```
4. 기존 테이블에서 기존 열 삭제
```sql
ALTER TABLE 기존테이블명
    DROP (열이름, ...);
--> 기존 테이블에 기존 데이터가 같이 삭제된다. 복구 불가.
```
5. 기존 테이블 이름 변경
```sql
RENAME 기존테이블명 TO 새로운테이블명;
```
6. 기존 테이블 삭제 (휴지통 기능 있음)
```sql
DROP TABLE 테이블이름;
--> 테이블 삭제시 관련 객체(제약조건, 인덱스, 트리거 등)들이 같이 삭제된다.
--> 참조 관계가 있는 테이블은 제약조건에 따라서 삭제 안되는 경우가 있다.
--> 단독 테이블은 항상 삭제 가능.
--휴지통에 있는 객체 확인
SELECT *FROM recyclebin;
--휴지통에 있는 객체 복원 (테이블명은 휴지통 내에서 부여된 임시 객체명)
FLASHBACK TABLE 테이블명 TO BEFORE DROP;
--휴지통 비우기
PURGE recyclebin;
-- DROP TABLE 테이블이름 PURGE;
SELECT * FROM recyclebin;
--> 삭제한 테이블 객체 확인 가능
FLASHBACK TABLE "BIN$GN3ZdqRIRwa9C4GaGAWpYg==$0" TO BEFORE DROP;
SELECT * FROM user_tables;
--> test1 테이블 존재 확인
SELECT * FROM test1;
--> 기존 자료 확인
--휴지통 비우기
PURGE recyclebin;
--> 복원 불가
--휴지통 기능을 거치지 않고 직접 삭제
DROP TABLE test1 PURGE;
--> 복원 불가
```
7. 제약 삭제
```sql
ALTER TABLE 테이블명
DROP CONSTRAINT (참조)제약명;
--> 잘못 지정된 제약을 수정하려면 삭제 후 추가해야 한다.
```

### DML 구문에 사용될 수 있는 용어에 대해 알아보자.

 DML(Database Manipulation Language)는 말 그대로 데이터베이스 조작 구문으로, 조건을 통한 데이터베이스 객체의 데이터 검색, 삽입, 변경, 삭제 작업을 가능케 하는 쿼리문이라고 할 수 있다.

- SELECT
지난 포스팅에서 계속해서 알아보았던, 검색용 질의문이다.
- INSERT
INSERT 문은 테이블에 새 행(row)을 추가하는데 이용하며, single table insert이나 multi table insert를 수행할 수 있다.
- UPDATE
테이블에서 기존의 데이터를 변경한다.
- DELETE
테이블에서 지정한 행을 삭제하는데 사용한다.

#### 쿼리문 설명에 앞서, DEFAULT 표현식에 대해 간단하게 정리하고 알아보도록 하자.
- insert와 update 문에서 특정 값이 아닌 디폴트값을 입력할 수도 있다.
- 형식: 컬럼명 데이터타입 DEFAULT 디폴트값
- INSERT 명령 실행시 해당 컬럼에 값을 할당하지 않거나, DEFAULT 키워드에 의해서 디폴트값을 입력할 수 있다.
- DEFAULT 키워드와 다른 제약 (NOT NULL 등) 표기가 같이 오는 DEFAULT 키워드를 먼저 표기할 것!
```sql
CREATE TABLE bbs (
 sid NUMBER PRIMARY KEY  --글번호(자동 번호 증가, 자동 입력)
 ,name VARCHAR2(10)      --글쓴이 이름
 ,content VARCHAR2(100)  --글내용
 ,writeday DATE  DEFAULT SYSDATE  --글쓴 날짜(현재 날짜 자동 입력)
);
--테이블 생성 후 DEFAULT 표현식 추가
ALTER TABLE 테이블명
 MODIFY 컬럼명 [자료형] DEFAULT 값;

--DEFAULT 표현식 삭제
ALTER TABLE 테이블명
 MODIFY 컬럼명 [자료형] DEFAULT NULL;
```

#### INSERT 쿼리문에 대해 알아보자.
- Single table insert : 오직 하나의 테이블이나 뷰에 오직 하나의 행(row)의 값들을 삽입할 수 있다.
- Multi table insert : 하나 이상의 테이블로부터 서브 쿼리로 얻은 여러 행(row)을 삽입하는 경우이다.
```sql
INSERT INTO 테이블_명 [(컬럼_명1, 컬럼_명2, ...)] VALUES (값1, 값2, ...);
-- 주의) 컬럼명과 값은 서로 일치(갯수, 순서, 자료형)해야 한다.
-- 주의) INSERT 명령 실행시 메모리에서만 입력된 상태이므로, 물리적 저장이 필요하면 COMMIT; 명령 실행 필요.
```

#### UPDATE 쿼리문에 대해 알아보자.
```sql
UPDATE 테이블_명
 SET 컬럼_명= 변경할_값[, 컬럼_명= 변경할_값, ...]
 [WHERE 조건]; --주의) WHERE 조건절을 지정하지 않거나, 잘못된 조건인 경우 원하지 않는 row까지 수정의 범위에 포함된다.

--서브쿼리를 UPDATE 구문의 일부로 사용 가능
UPDATE 테이블명
    SET 컬럼명 = (서브쿼리)
    WHERE (서브쿼리를 이용한 조건식);

UPDATE (서브쿼리-JOIN 쿼리를 이용한 가상 테이블)
    SET 컬럼명 = 값
    WHERE 조건식;
```

#### DELETE 쿼리문에 대해 알아보자.
```sql
DELETE [FROM] 테이블_명 [WHERE 조건];
-- 주의) WHERE 조건절을 지정하지 않거나, 잘못된 조건인 경우 원하지 않는 row까지 삭제의 범위에 포함된다.
--> FK 제약 생성시 ON DELETE CASCADE 옵션 지정하면 삭제 가능
```

### DCL 구문에 사용될 수 있는 용어에 대해 알아보자.

 DCL(Database Control Language)는 말 그대로 데이터베이스  제어 구문으로, 데이터베이스 접근 영역에 대한 권한을 부여하거나 부여된 권한을 취소하는 작업을 가능케 하는 쿼리문이다.

- GRANT
데이터베이스와 관련된 각종 권한을 부여한다.
(계성 생성, 계정 삭제, 테이블 삭제, 데이터베이스 접속, 테이블 생성, 뷰 생성, 시퀀스 생성, 함수 생성 등의 권한이 있다.)
- REVOKE
부여된 권한을 취소한다.

 오라클에서 권한을 설정하기 위해 sqldeveloper의 GUI를 활용해도 되지만, 여기선 쿼리문을 더 살펴보도록 하자.

#### GRANT  쿼리문에 대해 알아보자.
```sql
-- 계정 생성
CREATE USER 계정명 IDENTIFIED BY 비밀번호;
-- 데이터베이스 접속 권한 부여
GRANT CREATE 권한명 (SESSION, CONNECT, etc..)
TO 계정명
WITH ADMIN OPTION; -- 이 옵션을 통해 다른 사용자에게 해당 계정이 똑같이 권한을 줄 수가 있다.

-- 객체 권한 생성
GRANT 권한명, ...(SELECT, UPDATE, DELETE, INSERT etc...)
ON 객체명(테이블명) TO 계정명;

-- 부여한 권한 확인
SELECT * FROM USER_TAB_PRIVS_MADE;
-- 부여받은 권한 확인
SELECT * FROM USER_TAB_PRIVS_RECD;
```
#### REVOKE  쿼리문에 대해 알아보자.
```sql
-- 객체 권한 취소
REVOKE 권한명, ...(SELECT, UPDATE, DELETE, INSERT etc...)
ON 객체명(테이블명) FROM 계정명;
```

이렇게 간단하게? 오라클 제약조건 및 DDL, DCL, DML에 대해 알아보았다. 역시 데이터베이스 정리는 다른 부분보다 내용이 길어지는? 경향이 있다. 그 만큼 중요하고, 열심히 해야하는 이유일 듯 싶다. 헿.  

내용이 많이 부실하지만, 다음 포스팅에서 더 보완해보자.. ~~귀차니즘..~~

[다음 포스팅](https://seongjaemoon.github.io/2018/02/18/database-oracle5/)에선 뷰와 오라클 정리를 해보는 걸로~

* 오타나 잘못된 부분을 지적 해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* 참고문헌<br>
서진수, 김도균 지음 다양한 예제로 쉽게 배우는 오라클 SQL과 PL/SQL.<br>
미크 지음, 윤인성 옮김 DB 성능 최적화를 위한 SQL 실전 가이드 SQL 레벨업.
