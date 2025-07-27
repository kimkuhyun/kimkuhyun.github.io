# Oracle 실습: Advanced 과정 실습 정리

이번 주 실습은 SELECT 구문 작성 능력을 기반으로, 실무에서 성능 중심으로 활용되는 ORACLE SQL의 고급 기능을 학습하는데 중점을 두었습니다.
특히, JOIN과 SUBQUERY 를 다양한 방식으로 실습하며, 실행 계획을 통한 성능,분석,인덱스 활용 여부 판단, INLINE VIEW/WITH절, 집계 및 분석
함수 등 쿼리 튜닝과 구조화 능력에 대한 소개와 그것들을 경험했습니다.

## 주요 학습 주제 요약

- 테이블 간 관계 이해 : 외래키 기반 조인 관계 설정 및 엔터티 간 역할 파악
- JOIN 유형 별 비교 실습 : INNER, OUTER 등의 사용
- 서브쿼리 구조 학습 : 단일행, 다중행, 상관 서브쿼리 , WITH절 등
- 인덱스가 작동하는/작동하지 않는 조건 일부 실습
- 실행 계획 일부 확인
- 분석 함수 실습 : RANK, ROW_NUMBER,LAG, 등의 윈도우 함수 실습
- 성능 향상을 위한 힌트 일부 확인

---

## 실습 내용 및 예제

### INNER JOIN

-두 테이블의 교집합을 반환 
```SQL
SELECT E.NAME, D.DNAME
FROM EMP E
JOIN DEPT D ON DEPTNO = DEPTNO;
```

### OUTER JOIN(LEFT / RIGHT / FULL)

-한쪽 테이블의 모든 행을 유지하고, 매칭 안되는 값은 NULL 처리
```SQL
SELECT e.ename, d.dname
FROM emp e
LEFT JOIN dept d ON e.deptno = d.deptno;

SELECT e.ename, d.dname
FROM emp e, dept d
WHERE e.deptno = d.deptno(+);
```

### 서브쿼리 활용 

- FROM절 (인라인 뷰)
  - 결과가 테이블에 보여야 하고, 조건도 줄 수 있는 구조 (결과를 테이블처럼 활용)
```SQL
SELECT *
FROM (
  SELECT empno, sal FROM emp
) sub
WHERE sub.sal > 3000;
```
- WHERE절
  - 결과를 조건에만 사용하고 화면 출력엔 포함되지 않을 때 (조건 비교용)
```SQL
SELECT ename, sal
FROM emp
WHERE sal > (SELECT AVG(sal) FROM emp);
```
- SELECT절
  - 개별 행에 대해 관련 정보 보여줄 때 (온라인에서 유용)
```SQL
SELECT ename,
       (SELECT COUNT(*) FROM emp mgr WHERE mgr.mgr = emp.empno) AS num_subordinates
FROM emp;
```

### WITH절 

- 서브쿼리의 가독성과 재사용이 목적
  - 쿼리에 따라 프로세스가 임시로 테이블을 만들수도 있고 인라인 뷰로 변환만 할 수 도 있음 
```SQL
WITH dept_avg AS (
  SELECT deptno, AVG(sal) AS avg_sal
  FROM emp
  GROUP BY deptno
)
SELECT e.ename, e.sal, d.avg_sal
FROM emp e
JOIN dept_avg d ON e.deptno = d.deptno
WHERE e.sal > d.avg_sal;
```

### 인덱스

- 인덱스는 소량(테이블의 1~3% 수준) 데이터 조회 시 유리
- 인덱스는 인덱스 테이블에 저장된 행과 열의 주소값을 통해 실제 데이터를 참조함 (주소값은 디스크 → 블록 → 섹터 기준) 
- 함수 가공 컬럼, 암시적 형변환 등은 인덱스 미사용
- 비교 연산자 성능: = > BETWEEN > 부정( != , NOT IN )
- 인덱스가 쓰이는 문장을 쓸 줄 알아야 인덱스를 썼을 때와 안썼을 때 성능 차이를 구분 할 수 있어 인덱스가 쓰이는 문장을 써야한다고 함

```SQL
-- 인덱스가 작동하는 예시
SELECT *
FROM employees
WHERE commission_pct = 0.3;

-- 인덱스가 작동하지 않는 예시
SELECT *
FROM employees
WHERE SUBSTR(ename,1,1) = 'A';
```

### 그룹화 함수

- 그룹화 함수와 계층질의는 현재 단계에선 중요한 것이 아니라 함수의 소개 정도만 진행.

```SQL
SELECT deptno, job, SUM(sal)
FROM emp
GROUP BY ROLLUP(deptno, job);
```
- ROLLUP: 부분합 생성
- CUBE: 모든 조합 생성
- GROUPING SETS: 특정 조합만 집계
  
### 계층질의 

```SQL
SELECT LEVEL, empno, ename, mgr
FROM emp
START WITH mgr IS NULL
CONNECT BY PRIOR empno = mgr;
```

---

## 실습 총평
- 이번 실습에서 조인과 서브쿼리 개념 및 활용법을 집중 학습하여 쿼리 구조 이해도를 높였고, 실행 계획·인덱스에 따른 성능 차이를 체험하면서 SQL 최적화의 중요성을 확인했습니다. 단순히 결과를 얻는 것을 넘어서, 더 빠르고 효율적으로 데이터를 조회하는 방법을 고민하는 역량을 키웠으며, 향후 실무에서도 구조화·성능 중심의 고급 SQL 작성이 가능할 기반을 마련했습니다.
























