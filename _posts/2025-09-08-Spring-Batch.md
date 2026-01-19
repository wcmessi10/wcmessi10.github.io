---
layout: post
title: Spring Batch를 활용한 데이터 집계
tags: [Spring Boot, Spring Batch, Scheduling, Backend]
comments: true
mathjax: true
author: 이희두
---

> 본 문서에서 “스테이션”은 배터리 충전이 이루어지는 물리적 거점 또는 데이터 집계를 위한 논리적 단위를 의미한다.

# 개발 목표

탄소 감축량 산정에 필요한 **배터리 충전 로그 테이블**을 활용하여  
계산된 **스테이션 별 일별 배터리 충전량**을 별도 집계 테이블에 기록하는 로직을  
**특정 로컬 시간대 기준 오전 6시**에 스케줄링하여 자동 실행한다.

---

# 활용 기술

- Spring Batch  
- Async

---

# Spring Batch란?

**대용량 데이터를 처리하기 위한 프레임워크로, 스프링 프레임워크 기반에서 작동한다.  
일반적으로 배치 작업은 대량의 데이터를 처리하거나, 주기적이고 반복적인 작업을 실행하는 데 사용된다.**

---

## Spring Batch 구성 요소

### ✅ Job

- 배치 단위 작업의 시작점
- 예시: `dailyStationKwhJob`

### ✅ Step

- Job의 하위 단위 작업
- 예시: `calculateKwhStep`
- 하나의 Step에서 Tasklet 실행

### ✅ Tasklet

- 단일 기능성 처리 로직 실행
- 각 스테이션 ID에 대해 kWh 적산 계산 및 저장 수행

### 주의사항

- 배치 메타데이터 테이블이 자동 생성되지 않아  
  Spring Batch 라이브러리에서 제공하는 **메타데이터 테이블 생성 SQL**을  
  yml 설정을 통해 명시적으로 호출함

---

# 프로세스

### 단일 호출

- 수동 실행: `JobLauncher`를 통해 트리거
- API 호출 기반 수동 실행 지원  
  (예: 내부 스케줄러 관리용 endpoint)

---

### 스케줄러 연동

- `@Scheduled(cron = "0 23 * * * *")`
  - UTC 기준으로 특정 로컬 시간대 오전 6시에 실행
- 실행 환경 제한:
  - `@Profile("prod")`를 사용하여 운영 환경에서만 스케줄 실행

---

### 병렬 처리 구조

- 스테이션별 ID를 기준으로 내부 병렬 처리
  - `@Async + CompletableFuture` 활용
- 외부 응답과 무관하게 내부에서는 비동기적으로 처리 진행

---

# 환경 설정

### build.gradle 의존성 추가

```gradle
implementation 'org.springframework.boot:spring-boot-starter-batch'
````

---

### yml 설정 예시 (RDBMS 기준)

```yaml
spring:
  sql:
    init:
      platform: mysql
      schema-locations: classpath:org/springframework/batch/core/schema-mysql.sql
      mode: never
```

* `mode: always`

  * 초기 개발 환경에서만 사용
* 운영 환경에서는 `never`로 설정하여
  메타데이터 테이블 중복 생성 방지

---

# 병렬 처리 구조를 선택한 이유

## 스테이션 일별 배터리 충전량 기록 프로세스

* 스테이션별 일별 배터리 충전량 기록 작업을 **빠르고 안정적으로 수행**하기 위해 비동기 처리 방식 도입
* 병렬 실행을 통해 각 스테이션 데이터를 **독립적으로 처리** 가능
* 대량 스테이션 환경에서도 처리 시간 단축
* 각 스테이션의 배터리 충전량은 **사전에 정의된 계산 공식**을 기반으로 산정

---

# 결과 및 기대 효과

* 배터리 충전량 계산 로직이 스케줄링되어 매일 안정적으로 실행
* 스테이션 단위 병렬 처리로 대용량 데이터 처리 성능 향상
* 탄소 감축량 산정을 위한 기초 데이터의 **정합성 및 신뢰성 확보**

```

---
