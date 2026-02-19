---
layout: post
title: Transcation Isolation Level
tags: [Consistency, Transaction, Lock, Isolation]
comments: true
mathjax: true
author: 이희두
---


## Transaction Isolation Level이란?

여러 트랜잭션의 동시성 제어에서 각 트랜잭션 간 어느 정도의 고립을 가질 지 나타내는 것이다. 

락을 어떻게 얼마나 사용할지에 대한 정책이다.

데이터 일관성, 성능을 조율할 수 있는 공식 수준이다.

## 격리 수준을 정리하는 이유

서비스에서 데이터 일관성과 데이터베이스 성능은 매우 중요하다. 일관성이나 성능이 기준치에 맞지 않으면 서비스의 신뢰도는 급격하게 무너진다. 

데이터의 사용 빈도에 따라 적절한 격리 수준을 적용해야한다.

격리 수준을 정리하면서, 좀 더 서비스의 품질을 향상시킬 수 있는 레벨이 되는 것이 목적이다.

## 트랜잭션 관련 이상 현상

동시 실행되는 트랜잭션 간 상호작용에서 이상 현상이 발생할 수 있다

1. Dirty Read (공식 정의)
    1. 다른 트랜잭션이 커밋하지 않은 데이터를 읽는다.
2. Non-Repeatable Read (공식 정의)
    1. 같은 쿼리를 두 번 읽었을 때 중간 수정 등 이유로 각 값이 달라진다.
3. Phantom Read (공식 정의)
    1. 같은 조건 조회를 했는데 중간에 다른 트랜잭션으로 인해 행의 개수가 달라진다.
4. Lost Update
    1. 동시 Update 트랜잭션으로 특정 Update가 취소되는 현상
5. Write Skew
    1. 각 다른 row를 수정하여 제약 조건을 위반하는 케이스

## 표준 Isolation Level

ANSI SQL과 ISO SQL 표준에 따른 Transaction Isolation Level이 존재한다.

1. READ UNCOMMITTED
    1. 가장 낮은 격리 수준이다. 다른 트랜잭션들이 아직 커밋하지 않은 데이터도 접근할 수 있다.
    2. 허용하는 이상현상 
        1. Dirty Read
        2. Non-Repeatable Read
        3. Phantom Read
        4. Lost Update
    3. 성능은 가장 빠르다
2. READ COMMITTED
    1. 커밋된 데이터만 읽을 수 있다.
    2. 허용하는 이상 현상
        1. Non-Repeatable Read
        2. Phantom Read
    3. 가장 현실적인 기본 격리 수준이다.
    4. 조회 시 최신 커밋 데이터 확인이 가능하다.
    5. 대부분 서비스에서 많이 사용한다.
3. REPEATABLE READ
    1. 한 트랜잭션 안에서 같은 데이터를 여러번 읽어도 항상 같은 값이 보장된다.
    2. 허용하는 이상 현상
        1. Phantom Read
    3. DB 구현에 따라 Phantom도 막을 수 있다.
    4. 처음 조회 시점 스냅샷 유지
4. SERIALIZABLE
    1. 가장 높은 격리 수준
    2. 트랜잭션을 순차적으로 실행한 대로 결과값을 보장할 수 있다.
    3. 허용하는 이상 현상
        1. 없음
    4. 가장 안전하다
    5. 트랜잭션이 보유한 시간에 따라 락 경합이 심하다
    6. 성능 저하가 높다
    7. 가장 정합성이 높아야하는 로직에 필요하다

## Spring에서 적용하는 Transaction Isolation Level

@Transactional

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void test() {
    ...
}

@Transactional(isolation = Isolation.READ_UNCOMMITTED)
@Transactional(isolation = Isolation.REPEATABLE_READ)
@Transactional(isolation = Isolation.SERIALIZABLE)
@Transactional(isolation = Isolation.DEFAULT) // DB 기본값에 따른다

```

## 결론

Transaction Isolation Level은 동시성 환경에서 데이터 보호 수준을 결정하는 선택지다.

- 무조건 높은 격리 수준이 정답은 아니다.
- 데이터 특성과 트래픽 패턴에 맞춰 선택해야 한다.
- 필요하면 Lock 전략과 함께 사용해야 한다.

실무 기준:

- 일반 서비스 → READ COMMITTED
- 금액/재고/결제 → Lock 또는 높은 격리 수준 고려

격리 수준을 이해하는 것은 결국 **신뢰 가능한 서비스를 설계하는 기반**이 된다.
