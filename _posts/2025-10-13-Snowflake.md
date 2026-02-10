---
layout: post
title: Snowflake 알고리즘
tags: [Snowflake, Distributed Database, Primary Key]
comments: true
mathjax: true
author: 이희두
---

## 1. Snowflake 알고리즘이란?

분산 관계형 DB에서 고유한 64비트의 ID를 생성하는 알고리즘

- [1비트][41비트: 타임스탬프][10비트: 노드 ID][12비트: 시퀀스 번호]

## 2. Snowflake를 사용하는 이유

- 유니크, 시간 기반 순차성, 분산 환경에서의 높은 성능
- 높은 생성 속도

## 3. Snowflake 구현

```java
package kuke.board.common.snowflake;

import java.util.random.RandomGenerator;

public class Snowflake {
	private static final int UNUSED_BITS = 1;
	private static final int EPOCH_BITS = 41;
	private static final int NODE_ID_BITS = 10;
	private static final int SEQUENCE_BITS = 12;

	private static final long maxNodeId = (1L << NODE_ID_BITS) - 1;
	private static final long maxSequence = (1L << SEQUENCE_BITS) - 1;

	private final long nodeId = RandomGenerator.getDefault().nextLong(maxNodeId + 1);
	// UTC = 2024-01-01T00:00:00Z
	private final long startTimeMillis = 1704067200000L;

	private long lastTimeMillis = startTimeMillis;
	private long sequence = 0L;

	public synchronized long nextId() {
		long currentTimeMillis = System.currentTimeMillis();

		if (currentTimeMillis < lastTimeMillis) {
			throw new IllegalStateException("Invalid Time");
		}

		if (currentTimeMillis == lastTimeMillis) {
			sequence = (sequence + 1) & maxSequence;
			if (sequence == 0) {
				currentTimeMillis = waitNextMillis(currentTimeMillis);
			}
		} else {
			sequence = 0;
		}

		lastTimeMillis = currentTimeMillis;

		return ((currentTimeMillis - startTimeMillis) << (NODE_ID_BITS + SEQUENCE_BITS))
			| (nodeId << SEQUENCE_BITS)
			| sequence;
	}

	private long waitNextMillis(long currentTimestamp) {
		while (currentTimestamp <= lastTimeMillis) {
			currentTimestamp = System.currentTimeMillis();
		}
		return currentTimestamp;
	}
}
```

- **UNUSED_BITS (1)**
    
    → 64bit 정수에서 부호 비트(0으로 고정)
    
- **EPOCH_BITS (41)**
    
    → 기준 시점(epoch)으로부터 지난 시간을 표현할 비트 수.
    
    41bit면 약 69년 동안 밀리초 단위로 시간 표현 가능.
    
    순차성 보장
    
- **NODE_ID_BITS (10)**
    
    → 노드 ID(서버나 워커 구분)를 표현할 비트 수.
    
    2¹⁰ = 1024개의 노드를 표현할 수 있음.
    
    동일한 시간에서 순차성 보장
    
- **SEQUENCE_BITS (12)**
    
    → 같은 밀리초에 생성되는 시퀀스 번호를 표현할 비트 수.
    
    2¹² = 4096개의 ID를 1ms 안에 생성 가능.
    
    동일한 시간에서 순차성 보장
    
- `maxNodeId`: 가능한 노드 ID 최대값 (1023)
- `maxSequence`: 가능한 시퀀스 최대값 (4095)

비트 수로 표현할 수 있는 최대값을 계산.

### 순서

1. 현재 시간 읽기
2. 시간 역행 체크
3. 같은 밀리초 내 요청인지 체크
4. sequence 오버플로 체크
    1. (같은 밀리초인가?)
    ├─ 예: sequence++
    │    └─ sequence 가득 찼으면 → 다음 밀리초까지 대기
    └─ 아니오: sequence = 0
5. 마지막 생성 시간 갱신
6. ID 조합 (timestamp + nodeId + sequence)
