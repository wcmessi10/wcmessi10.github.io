---
layout: post
title: Data Collation
tags: [Database, Character Encoding, Sorting, Comparing]
comments: true
mathjax: true
author: 이희두
---

## Database Collation이란?

Character(문자열)를 정해진 인코딩을 기반으로 어떻게 비교, 정렬, 대소문자/악센트 처리할지를 정하는 규칙

### Example

대소문자 구분 : A와 a를 같은 걸로 볼까

악센트 구분 : é랑 e를 같은 것으로 볼까

정렬 순서 : 가, 나, 다 / a, b, c를 어떤 규칙으로 정렬할까

비교 결과 : where name = ‘abc’가 ‘ABC’도 포함일까

## 많이 쓰는 Database Collation들

### utf8mb4_general_ci

- Case-insensitive (대소문자 구분 없음)
- 악센트 구분 없음
- 비교 속도 빠름 (정확도 낮음)
- 구형
- 한글/유럽어 정렬 미묘하게 틀릴 수 있음
- ddl-auto로 대충 만든 테이블에서 주로 사용

### utf8mb4_unicode_ci

- Case-insensitive (대소문자 구분 없음)
- 악센트 구분 없음
- Unicode 표준 기반 → 정렬 정확
- JPA + MySQL 조합
- 검색 키워드 용으로 주로 사용
- 실무 최다 사용

### utf8mb4_0900_ai_ci

- Case-insensitive (대소문자 구분 없음)
- 악센트 구분 없음
- MySQL 8.x 기본& 최신 표준 - Unicode 9.0 기반

### utf8mb4_unicode_cs

- Case-sensitive (대소문자 구분)
- 악센트 구분 없음
- 대소문자 구분용으로 사용

### utf8mb4_0900_as_cs

- 가장 엄격한 문자열 비교
- Case-sensitive (대소문자 구분)
- 악센트 구분 있음

### utf8mb4_bin

- Case-sensitive (대소문자 구분)
- 악센트 구분
- 문자열의 코드 포인트 값을 바탕으로 비교
- 해시 / 암호 전용

## Collation의 사용법

Collation은 여러 레벨에서 사용가능

### 1. Database

```sql
CREATE DATABASE mydb
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
-- Database 딴에서 생성
-- 테이블 or 컬럼에 collation을 명시하지 않으면 이 설정이 적용된다

ALTER DATABASE mydb
COLLATE utf8mb4_unicode_ci;
-- 수정
-- 이미 만들어져있는 테이블에는 영향이 없다
```

### 2. Table

```sql
CREATE TABLE user (
  id INT PRIMARY KEY,
  name VARCHAR(50)
)
CHARACTER SET utf8mb4
COLLATE utf8mb4_0900_ai_ci;
-- 테이블 생성 시점 해당 테이블의 모든 컬럼 적용
-- Database 설정된 Character set보다 우선순위 높다
```

### 3. Column

```sql
name VARCHAR(50)
COLLATE utf8mb4_unicode_cs
-- 컬럼 별로 설정 가능
-- 가장 우선순위가 높다
-- 실무에서 많이 활용
```

### 4. Query

```sql
SELECT *
FROM user
WHERE name COLLATE utf8mb4_unicode_ci = 'abc';

SELECT *
FROM user u
JOIN order o
  ON u.user_id COLLATE utf8mb4_unicode_ci
   = o.user_id COLLATE utf8mb4_unicode_ci;
-- 일회성으로 해당 쿼리만 적용
-- 인덱스를 안 탈 수 있음
-- 성능 저하 이슈 가능
```

## Collation 관련 이슈

### 조인 테이블 문자열 컬럼 collation 차이 관련 이슈

조인 테이블의 join하는 테이블 별 조인 컬럼들의 collation이 다른 경우 오류 발

```sql
-- 테이블 A
user_id VARCHAR(50)
COLLATE utf8mb4_unicode_ci

-- 테이블 B
user_id VARCHAR(50)
COLLATE utf8mb4_0900_ai_ci

SELECT *
FROM user u
JOIN order o ON u.user_id = o.user_id;

Illegal mix of collations (utf8mb4_unicode_ci, utf8mb4_0900_ai_ci)

```

해결 방안

- 컬럼 collation 통일
- Query로 collation 통일 - 임시방편

### UNIQUE 컬럼에 ci 사용할 경우 생기는 대소문자 이슈

시스템에서는 대소문자 차이로 시스템적으로는 다른 값인데 DB에서는 같은 값으로 판단하여 오류 발생

```sql
Duplicate entry

```

해결방안

- ci가 아닌 cs로 UNIQUE 컬럼값 적용

### JPA에서 ddl-auto : update 관련 이슈

ddl-auto : update로 설정 후 Hibernate에서 collation을 명시하지 않은 경우 기본값으로 자동 적용된다

해결 방안

- 운영 환경 ddl-auto : none 처리
- 엔티티 클래스의 컬럼에 collation 명시
- DB 생성 시 기본값 고
