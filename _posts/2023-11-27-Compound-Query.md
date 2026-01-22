---
layout: post
title: Compound Query
tags: [Elasticsearch, QueryDSL, Compound Query]
comments: true
mathjax: true
author: 이희두
---


## 복합 쿼리 (Compound Query)

**복합 쿼리(Compound Query)** 는
Elasticsearch에서 **여러 개의 리프 쿼리(Leaf Query)를 논리적으로 조합하여 하나의 검색 조건을 구성하는 쿼리**입니다.

* 리프 쿼리: 단일 필드 + 단일 연산
* 복합 쿼리: **여러 리프 쿼리를 AND / OR / NOT 구조로 조합**

실제 서비스에서 사용하는 검색 쿼리의 대부분은 **복합 쿼리** 형태입니다.

---

## Query Context vs Filter Context

Compound Query를 이해하기 위해 가장 중요한 개념이 바로 **Context(컨텍스트)** 입니다.

Context를 한글로 풀면 **“절”**에 가깝습니다.

### 두 가지 컨텍스트

| 구분       | Query Context     | Filter Context   |
| -------- | ----------------- | ---------------- |
| 목적       | 관련성(Relevance) 계산 | 일치/불일치 판단        |
| Score 계산 | O                 | X                |
| 캐시 사용    | X                 | O                |
| 주 사용 쿼리  | Full-text query   | Term-level query |

---

### Query Context (Query 절)

* **Relevance Score(_score)** 를 계산
* 검색 결과의 **정렬 순위**에 영향
* 주로 **전문 검색(full-text query)** 사용

예:

* `match`
* `match_phrase`
* `multi_match`

---

### Filter Context (Filter 절)

* **Score 계산 없음**
* 단순히 **조건 일치 여부(true / false)** 만 판단
* 결과 캐싱 가능 → 성능상 유리
* 주로 **정확한 값 비교 쿼리(term-level)** 사용

예:

* `term`
* `terms`
* `range`
* `exists`

**조건 필터링은 filter**,
**검색어 유사도는 query**

이게 실무에서 가장 중요한 원칙입니다.

---

## bool query

### 개요

`bool query`는 가장 대표적인 **Compound Query**이며,
여러 쿼리를 논리적으로 조합할 수 있는 4가지 절을 제공합니다.

---

### bool query의 4가지 절

| 절        | 컨텍스트   | 설명                     |
| -------- | ------ | ---------------------- |
| must     | Query  | 모두 만족해야 일치             |
| must_not | Query  | 모두 불일치해야 일치            |
| should   | Query  | 많이 일치할수록 score 증가      |
| filter   | Filter | 모두 만족해야 일치 (score 미반영) |

---

### must

* 지정된 모든 쿼리가 **true**
* 관련성 점수 계산에 포함

```json
{
  "must": [
    { "match": { "name": "Messi" } }
  ]
}
```

---

### must_not

* 지정된 모든 쿼리가 **false**
* 결과에서 제외 조건

```json
{
  "must_not": [
    { "term": { "status": "INACTIVE" } }
  ]
}
```

---

### should

* 일치하는 쿼리가 많을수록 **_score 증가**
* 검색 결과 **순위 조정** 용도

```json
{
  "should": [
    { "match": { "description": "football" } },
    { "match": { "description": "legend" } }
  ]
}
```

---

### filter

* 모든 쿼리가 true여야 일치
* **Score 계산 X**
* 캐시 가능 → 성능 최적화에 중요

```json
{
  "filter": [
    { "term": { "status": "ACTIVE" } },
    { "range": { "age": { "gte": 30 } } }
  ]
}
```

---

### bool query 전체 예시

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "Messi" } }
      ],
      "filter": [
        { "term": { "status": "ACTIVE" } },
        { "range": { "age": { "gte": 30 } } }
      ]
    }
  }
}
```

`must` → 검색어 관련성
`filter` → 조건 필터링

---

## Boosting Query

### 개요

**boosting query**는

> *특정 조건을 만족하는 문서의 점수(score)를 “낮추는” 데 특화된 복합 쿼리*입니다.

**제외하지는 않되, 우선순위를 낮추고 싶을 때** 사용합니다.

---

### boosting query 옵션

| 옵션             | 설명                 |
| -------------- | ------------------ |
| positive       | 반드시 일치해야 하는 쿼리     |
| negative       | 점수를 낮출 대상 쿼리       |
| negative_boost | 점수 감소 비율 (0 ~ 1.0) |

---

### boosting query 예시

```json
{
  "boosting": {
    "positive": {
      "match": {
        "content": "football"
      }
    },
    "negative": {
      "term": {
        "category": "advertisement"
      }
    },
    "negative_boost": 0.3
  }
}
```

광고 문서는 제거하지 않지만 **검색 순위에서 밀려남**

---

## (보충) function_score / component_score 관련

질문에 적어주신 `component_score query`는
보통 실무에서는 **`function_score` 쿼리**로 이어서 설명하는 경우가 많습니다.

* 점수(_score)를 수식으로 조정
* 가중치(weight), 스크립트, 필드 값 기반 점수 반영
* 랭킹, 추천, 중요도 반영에 사용

boosting이 “점수 감소” 중심이라면
function_score는 “점수 재계산/확장”에 가깝습니다.

(이건 다음 문서로 따로 빼는 게 깔끔합니다.)

---

## 실무 기준 요약

* **검색어 관련성** → `must`, `should` (Query Context)
* **필터 조건** → `filter` (Filter Context)
* **제외는 아님, 우선순위만 조정** → `boosting`
* **성능 최적화 핵심** → 가능한 한 `filter` 활용

---

## 한 줄 요약

> **Compound Query는 여러 리프 쿼리를 조합하는 구조이며,
> Query Context는 점수 계산, Filter Context는 조건 필터링을 담당한다.
> 실무에서는 bool query + filter 절 조합이 핵심 패턴이다.**
