---
layout: post
title: Record Lock
tags: [Database, Lock, Concurrency issue]
comments: true
mathjax: true
author: 이희두
---

## Lock(락)

여러 프로세스 또는 스레드가 자원에 동시에 접속하는 것(경쟁 상태)을 방지하기 위해 제한을 거는 것

## Record Lock

- 테이블의 로우에 락을 거는 것
- 동일한 로우에 동시 조회 or 수정할 때, 데이터의 무결성 보장, 경쟁 상태 방지
- 락을 오래 점유하면 리소스 고갈 등 치명적이고 다양한 문제가 발생
- 비관적 락

레코드(Row)에 수행한 작업이 걸린 락을 조회하는 쿼리

select * from performance_schema_data_locks;

## Shared Lock (공유 락)

- **읽기(Read) 전용** 작업을 위한 락
- 여러 트랜잭션이 동시에 같은 데이터를 읽을 수 있음
- 하지만 **쓰기(Write) 작업은 불가능**
- 즉, **다른 Shared Lock은 허용**, **Exclusive Lock은 불허**

## Exclusive Lock (배타 락)

- **쓰기(Write) 전용** 작업을 위한 락
- 데이터를 변경하기 전에 반드시 획득해야 함
- **읽기, 쓰기 모두 차단**
- 다른 어떤 락도 동시에 잡을 수 없음 (Shared 포함)

## 1. 비관적 락 (Pessimistic Lock)

- **“동시 충돌이 자주 일어날 거다”** 라고 가정하고 미리 락을 걸어버리는 방식.
- 데이터에 접근할 때 바로 **DB에서 Shared Lock / Exclusive Lock** 을 획득하여 다른 트랜잭션이 건드리지 못하게 막음.
- 즉, **선(先)차단 → 후(後)작업**.
- 트랜잭션이 긴 경우, **Deadlock(교착상태)** 위험 있음.
- 충돌은 확실히 막을 수 있지만, **동시성 처리 성능이 떨어짐**.
- DBMS 레벨에서 `SELECT ... FOR UPDATE`, `SELECT ... LOCK IN SHARE MODE` 같은 방식으로 구현.
- Spring Boot에서는 @Lock(LockModeType.{락타})

## 2. 낙관적 락 (Optimistic Lock)

- **“충돌은 드물 것이다”** 라고 가정.
- 처음에는 락을 걸지 않고 데이터를 읽은 뒤, **커밋 시점에 변경 여부를 검증**.
- 일반적으로 **버전(version) 컬럼**이나 **타임스탬프**를 이용해 충돌 감지.
- 즉, **선(先)작업 → 후(後)검증**.
- 동시성이 높고 읽기 위주의 시스템에 유리.
- 충돌이 감지되면 트랜잭션을 **롤백 후 재시도**해야 함.
- JPA/Hibernate에서는 `@Version` 애노테이션으로 구현.

| 구분 | 비관적 락 (Pessimistic) | 낙관적 락 (Optimistic) |
| --- | --- | --- |
| 접근 방식 | 먼저 락을 걸고 안전하게 작업 | 일단 수행 후, 커밋 시점에 충돌 검증 |
| 충돌 처리 | DBMS가 충돌을 막음 | 충돌 발생 시 애플리케이션이 처리 |
| 장점 | 데이터 정합성 확실, 충돌 방지 | 동시성 처리 유리, 락 오버헤드 적음 |
| 단점 | 성능 저하, Deadlock 위험 | 충돌 시 재시도 필요 |
| 적합 환경 | **쓰기 많은 시스템** | **읽기 많은 시스템** |
