---
layout: post
title:  "Oracle 내장 함수 정리2"
date:   2018-01-14 23:00:00
author: Seongjae Moon
categories: DB
tags:   Oracle 오라클 함수
---

[저번 포스팅](https://seongjaemoon.github.io/2018/01/09/database-oracle1/)에서 오라클 DB의 내장 함수에 대해 간단하게 알아보았다. 하지만, 오라클에선 더 많은 내장 함수를 제공한다. 아직 갈 길이 멀다는 것을 알 수 있다...

이번 포스팅에선 오라클에서 제공하는 분석함수와 서브 쿼리에 대해 정리해보자.

## 분석함수
행끼리 연산이나 비교를 쉽게 지원해주기 위한 함수.
ROLLUP(),CUBE(),GROUPING SETS(),LISTAGG(),PIVOT(),LAG()

### ROLLUP() 함수
기준별(GROUP BY) 소계(COUNT, SUM)를 요약해서 보여주는 함수.
```sql
--hr.employees 테이블에서 부서번호(department_id)별로 인원수 출력
SELECT NVL(department_id,-1) department_id,COUNT(*)
FROM hr.employees
GROUP BY ROULLUP(NVL(department_id,-1))
ORDER BY department_id;
```
### CUBE() 함수
기준별(GROUP BY) 소계(COUNT, SUM)및 전체합계를 요약해서 보여주는 함수.
```sql
SELECT job_id,COUNT(*)
FROM hr.employees
GROUP BY CUBE(job_id)
ORDER BY job_id;

SELECT department_id,job_id ,COUNT(*)
FROM hr.employees
GROUP BY CUBE(department_id,job_id)
ORDER BY department_id,job_id;
```
### GROUPING SETS() 함수
기준별(GROUP BY) 소계(COUNT, SUM)및 전체합계를 요약해서 보여주는 함수.
```sql
SELECT department_id,job_id ,COUNT(*)
FROM hr.employees
GROUP BY GROUPING SETS(department_id,job_id)
ORDER BY department_id,job_id;
```
### LISTAGG() 함수
출력시 자료를 하나의 문자열로 통합 출력.
```sql
--hr.employees 테이블에서 부서번호별(department_id)별 인원수 및 명단(first_name) 출력.
SELECT department_id
    , COUNT(*)
    , LISTAGG (first_name, '/') WITHIN GROUP(ORDER BY first_name) first_names
FROM hr.employees
GROUP BY department_id;
```
### PIVOT() 함수
출력시 가로 형태를 세로 형태로 바꿔주는 함수.
```sql
SELECT department_id
    ,COUNT(*)
    ,COUNT(DECODE(job_id,'AD_ASST',1)) AD_ASST
    ,COUNT(DECODE(job_id,'MK_MAN',1)) MK_MAN
    ,COUNT(DECODE(job_id,'MK_REP',1)) MK_REP
FROM hr.employees
GROUP BY department_id;

SELECT *
FROM(SELECT employee_id, department_id,job_id FROM hr.employees)
PIVOT(
    COUNT(employee_id) for job_id IN('AD_ASST','MK_MAN','MK_REP')
)
ORDER BY department_id;
```
### LAG() 함수
이전 행 값을 가져오는 함수.
```sql
SELECT employee_id, first_name, last_name
    ,salary
    ,LAG(salary,1,0) OVER (ORDER BY salary DESC) LAG_
    ,salary - LAG(salary,1,0) OVER (ORDER BY salary DESC) LAG2_
FROM hr.employees
ORDER BY salary DESC;
```
### LEAD() 함수
이후 행 값을 가져오는 함수.
```sql
--hr.employees 테이블에서 급여(salary)를 출력하되, 이후 행의 급여(salary)를 같이 출력.
SELECT employee_id, first_name, last_name
    , salary
    , LEAD(salary, 1, 0) OVER (ORDER BY salary DESC) LEAD_
    , salary - LEAD(salary, 1, 0) OVER (ORDER BY salary DESC) LEAD2_
FROM hr.employees;
```
### RANK() 함수
전체 순위 부여 함수
주의) 동점자 처리 확인. 예를 들어, 1, 2, 2, 4 형태로 처리.
```sql
--hr.employees 테이블에서 급여(salary)가 높은 순에서 5번째
SELECT * FROM (SELECT employee_id, first_name, last_name
    , salary
    , RANK() OVER (ORDER BY salary DESC) rank_
FROM hr.employees)
WHERE rank_ <= 5;
```
### DENSE_RANK() 함수
전체 순위 부여 함수.
주의) 동점자 처리 확인. 예를 들어, 1, 2, 2, 3 형태로 처리.
```sql
SELECT employee_id, first_name, last_name
    , salary
    , DENSE_RANK() OVER (ORDER BY salary DESC) rank_
FROM hr.employees;
```
### ROW_NUMBER() 함수
고유 번호를 순번대로 부여하는 함수.
예를 들어, 1, 2, 3, 4 형태로 번호 부여.
```sql
SELECT employee_id, first_name, last_name
    , salary
    , ROW_NUMBER() OVER (ORDER BY salary DESC) rn_
FROM hr.employees;
```
### SUM() OVER() 함수
그룹별 누계를 출력해주는 함수.
```sql
SELECT SUM(salary)
FROM hr.employees
WHERE department_id = 90;

SELECT employee_id, first_name, last_name
    ,department_id
    , salary
    , (SELECT SUM(salary)
        FROM hr.employees
        WHERE department_id = emp.department_id) sum_
FROM hr.employees emp
ORDER BY employee_id;

SELECT employee_id, first_name, last_name
    ,department_id, salary
    , SUM(salary) OVER (PARTITION BY department_id) sum_
FROM hr.employees
ORDER BY employee_id;
```
### RATIO_TO_REPORT() 함수
그룹별 비중을 출력해주는 함수.
```sql
SELECT employee_id, first_name, last_name
    ,department_id, salary
    , SUM(salary) OVER (PARTITION BY department_id) sum_
    , ROUND((salary/SUM(salary) OVER (PARTITION BY department_id))*100, 1) "ratio_%"
FROM hr.employees
ORDER BY employee_id;

SELECT employee_id, first_name, last_name
    ,department_id, salary
    , SUM(salary) OVER (PARTITION BY department_id) sum_
    , ROUND(RATIO_TO_REPORT(salary) OVER (PARTITION BY department_id)*100, 1) "ratio_%"
FROM hr.employees
ORDER BY employee_id;
```
## 서브쿼리(SubQuery)
메인 쿼리 내에 SELECT 쿼리가 포함된 상태인 쿼리.
주의) 서브쿼리는 ()로 감싸야 한다.
서브쿼리가 먼저 실행되고, 결과가 나오면 메인쿼리가 실행된다.
```sql
--기본 형식
SELECT select_list
FROM table_list
WHERE 컬럼 연산자 (서브쿼리);
```
### 단일 행 서브쿼리
서브쿼리 결과가 1개의 행만 나오는 경우.
연산자는 비교 연산자(=, !=, >, <, >=, <=) 중에 하나 사용 가능.
```sql
--기본 형식
SELECT select_list
FROM table_list
WHERE 컬럼 = (결과값);
```
### 다중 행 서브쿼리
서브쿼리 결과가 여러개의 행이 나오는 경우.
연산자는 IN, EXISTS, > ALL,  > ANY, < ALL, < ANY 사용 가능.
```sql
--기본 형식
SELECT select_list
FROM table_list
WHERE 컬럼 IN (결과값1, 결과값2, ...);
```
### 다중 컬럼 서브쿼리
결과가 여러 컬럼으로 구성된 경우.
```sql
--기본 형식
SELECT select_list
FROM table_list
WHERE (컬럼1, 컬럼2, ...) IN (컬럼1_결과값, 컬럼2_결과값, ...);
```
### 상호 연관 서브쿼리
메인 쿼리의 결과를 가지고, 서브쿼리에 참여하고, 서브쿼리의 결과가 메인 쿼리에 참여하는 경우.
메인 쿼리의 테이블에 대해서 반드시 별칭 사용.
```sql
--기본 형식
SELECT select_list
FROM table_list 별칭
WHERE 컬럼 연산자 (서브쿼리의 조건식에 메인쿼리 컬럼 정보 참여);
```
### 스칼라 서브쿼리
메인 쿼리의 결과를 가지고, 서브쿼리에 참여하고, 서브쿼리의 결과가 메인 쿼리에 참여하는 경우.
메인 쿼리의 테이블에 대해서 반드시 별칭 사용.
OUTER JOIN과 같은 결과. JOIN 쿼리 권장.
```sql
--기본 형식
SELECT select_list
    , (서브쿼리의 조건식에 메인쿼리 컬럼 정보 참여)
FROM table_list 별칭;
```

간단하게? 오라클 내장 분석 함수 및 서브쿼리에 형식에 대해 정리해보았다. 조인(Join)만으로도 해결 가능한 경우가 많다고 하나, 분명 조인만으로는 특정 테이블의 값을 가져오는게 어려울 수 있다. 내장 함수와 적절한 서브쿼리를 잘 콜라보해서 값을 뽑아올 수 있도록, 쿼리문 날리는 연습을 많이 해야겠다. (**역시 연습만이 살 길 헿!**)

[다음](https://seongjaemoon.github.io/2018/01/27/database-oracle3/) 포스팅에선 RDB의 꽃 중 꽃인 '조인'과 테이블 '스키마'(Scheme)에 대해 알아보는 걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* 참고문헌<br>
서진수, 김도균 지음 다양한 예제로 쉽게 배우는 오라클 SQL과 PL/SQL<br>
미크 지음, 윤인성 옮김 DB 성능 최적화를 위한 SQL 실전 가이드 SQL 레벨업