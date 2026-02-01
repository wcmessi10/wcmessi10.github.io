---
layout: post
title: Elasticsearch Aggregation
tags: [Elastaicsearch, Aggregation]
comments: true
mathjax: true
author: 이희두
---


# Elasticsearch Aggregation 정리

Aggregation은
검색 결과를 분석하기 위한 기능입니다.

문서를 찾는 것이 아니라
데이터를 요약·통계·분석하는 것이 목적입니다.

SQL로 보면:

* GROUP BY
* SUM / AVG / COUNT
* HAVING
* 윈도우 함수

와 유사한 역할입니다.

---

# Bucket Aggregation (그룹화)

문서를 그룹으로 나누는 집계입니다.
SQL의 GROUP BY에 해당합니다.

각 bucket은 하나의 그룹입니다.

---

## 대표 종류

### 1) Terms Aggregation

가장 많이 사용하는 집계입니다.
특정 필드 값 기준 그룹화합니다.

```json
"aggs": {
  "by_status": {
    "terms": { "field": "status.keyword" }
  }
}
```

예시 결과:

```
SUCCESS: 1200
FAIL: 300
PENDING: 80
```

실무 예:

* 국가별 사용자 분포
* 에러 코드 집계

---

### 2) Date Histogram

시간 기준 그룹화입니다.

```json
"date_histogram": {
  "field": "reg_dt",
  "calendar_interval": "day"
}
```

활용 예:

* 시간대별 API 호출량
* 월별 매출

---

### 3) Range Aggregation

값 범위 기준 그룹화입니다.

```json
"ranges": [
  {"to": 20},
  {"from": 20, "to": 50},
  {"from": 50}
]
```

활용 예:

* 가격대별 분석

---

### 4) Histogram

숫자를 일정 간격으로 자동 분할합니다.

```json
"histogram": {
  "field": "pv",
  "interval": 5
}
```

예:
0~5, 5~10, 10~15 …

---

### 5) Filters Aggregation

여러 조건을 동시에 집계합니다.

```json
"filters": {
  "filters": {
    "full": { "term": {"status": "FULL"}},
    "empty": { "term": {"status": "EMPTY"}}
  }
}
```

서로 다른 조건 비교에 유용합니다.

---

# Metric Aggregation (통계 계산)

숫자 계산 집계입니다.
Bucket이 그룹이라면 Metric은 계산입니다.

---

## 대표 종류

### Sum

```json
"sum": {"field": "pa"}
```

전류 총합 계산.

---

### Avg

```json
"avg": {"field": "soc"}
```

평균 SOC 계산.

---

### Min / Max

최솟값 / 최댓값 계산.

---

### Stats

한 번에 제공:

* count
* min
* max
* avg
* sum

---

### Extended Stats

Stats + 분산 + 표준편차.

데이터 분포 분석에 유용합니다.

---

### Value Count (주의)

고유값 개수가 아니라
값이 존재하는 문서 개수입니다.

고유값 개수는
cardinality를 사용합니다.

---

### Percentiles

50%, 90%, 95%, 99% 등 백분위 계산.

활용 예:

* 응답시간 SLA 분석

---

### Top Hits

각 bucket에서 실제 문서 N개 반환합니다.

예:

* 최신 로그 1개
* 최고값 가진 문서

---

# Pipeline Aggregation (후처리 계산)

집계 결과를 다시 계산하는 집계입니다.
즉, 집계 위의 집계입니다.

---

## 대표 종류

### bucket_script

집계 결과로 수식 계산.

```json
"script": "params.a / params.b"
```

예:

* 성공률 계산

---

### derivative

변화율 계산.

예:

* 시간별 증가량
* 사용량 추이

---

### cumulative_sum

누적합 계산.

예:

* 누적 충전량
* 누적 사용자 수

---

### avg_bucket

여러 bucket 평균 계산.

---

### bucket_selector

조건 만족 bucket만 유지.
SQL의 HAVING과 유사.

---

### max_bucket

가장 큰 bucket 선택.

예:

* 최대 트래픽 시간대
* 최고 사용량 날짜

---

# 실무 핵심 포인트

### 1) DB 부하 감소

대량 데이터 분석을
애플리케이션이 아닌 Elasticsearch가 수행합니다.

---

### 2) 구조 이해

```
Bucket (그룹화)
   ↓
Metric (계산)
   ↓
Pipeline (후처리)
```

이 흐름을 이해하면 Aggregation을 제대로 사용할 수 있습니다.



