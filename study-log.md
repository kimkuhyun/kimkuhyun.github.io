# Oracle 실습: 관계형 DB 구현과 SQL 실습 흐름 정리

이번 주 실습을 통해 주어진 학사 시스템 구조를 기반으로 Oracle SQL을 이용한 관계형 데이터베이스 구현 및 운영 흐름을 학습했습니다.

## 주요 학습 내용

- 학사 시스템 구조에 따라 테이블 생성 (STUDENT, PROFESSOR, COURSE, LECTURE 등)
- 제약조건 설정: `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `CHECK`
- 시퀀스를 활용한 자동 학번 생성 (`STUDENT_SEQ`)
- 사용자 계정 생성 및 권한 분리
  - 교수: 성적 입력/조회
  - 학생: 본인 성적 조회 및 수강 신청
- 뷰(View) 생성
  - `V_STUDENT_GRADES`: 학생 본인 성적만 조회
  - `V_PROFESSOR_ALL_GRADES`: 교수는 전체 학생 성적 조회
- DML 문 실습: `UPDATE`, `DELETE` 시 외래 키 제약 조건 처리 경험
- 다양한 `SELECT` 쿼리를 활용한 조건별 조회, 집계, 필터링 실습

---

## 예시 쿼리 모음

### 1. 점수에 따라 Pass/Fail 구분 출력
```sql
SELECT s.student_name, g.score,
       CASE WHEN g.score >= 80 THEN 'Pass' ELSE 'Fail' END AS result
FROM grade g
JOIN enrollment e ON g.enroll_id = e.enroll_id
JOIN student s ON s.student_id = e.student_id;
```

### 2. 학생별 평균 성적과 총 학점 조회
```sql
SELECT s.student_id, s.student_name,
       SUM(c.credits) AS total_credits,
       ROUND(AVG(g.score), 1) AS avg_score
FROM student s
JOIN enrollment e ON s.student_id = e.student_id
JOIN lecture l ON e.lecture_id = l.lecture_id
JOIN course c ON l.course_id = c.course_id
JOIN grade g ON e.enroll_id = g.enroll_id
GROUP BY s.student_id, s.student_name;
```

### 3. 특정 과목을 수강한 학생 목록 + 평균 성적
```sql
-- 목록
SELECT s.student_name, g.score
FROM student s
JOIN enrollment e ON s.student_id = e.student_id
JOIN lecture l ON e.lecture_id = l.lecture_id
JOIN course c ON l.course_id = c.course_id
JOIN grade g ON e.enroll_id = g.enroll_id
WHERE c.course_name = '데이터베이스';

-- 평균
SELECT AVG(g.score) AS avg_score
FROM enrollment e
JOIN lecture l ON e.lecture_id = l.lecture_id
JOIN course c ON l.course_id = c.course_id
JOIN grade g ON e.enroll_id = g.enroll_id
WHERE c.course_name = '데이터베이스';
```

### 4. 가장 많은 강의를 맡은 교수 조회
```sql
SELECT p.prof_name, COUNT(l.lecture_id) AS lecture_count
FROM professor p
JOIN lecture l ON p.prof_id = l.prof_id
GROUP BY p.prof_name
ORDER BY lecture_count DESC
FETCH FIRST 1 ROWS ONLY;
```

### 5. 학과별 학생 수
```sql
SELECT d.dept_name, COUNT(s.student_id) AS student_count
FROM department d
LEFT JOIN student s ON d.dept_id = s.dept_id
GROUP BY d.dept_name;
```

### 6. 수강 신청 중복 방지 (복합 유니크키 설정)
```sql
CONSTRAINT uq_student_lecture UNIQUE (student_id, lecture_id)
```

### 7. 수강 인원이 가득 찬 강의 조회
```sql
SELECT l.lecture_id, l.max_capacity, COUNT(e.enroll_id) AS enrolled
FROM lecture l
JOIN enrollment e ON l.lecture_id = e.lecture_id
GROUP BY l.lecture_id, l.max_capacity
HAVING COUNT(e.enroll_id) >= l.max_capacity;
```
