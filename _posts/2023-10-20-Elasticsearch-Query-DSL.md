---
layout: post
title: ResponseEntity
tags: [Elasticsearch, QueryDSL]
comments: true
mathjax: true
author: 이희두
---

## Elasticsearch Query DSL 개요

**Query DSL (Domain Specific Language)** 은 Elasticsearch에서 **검색(Search)과 질의(Query)를 정의하기 위해 사용하는 JSON 기반 쿼리 문법**입니다.

Elasticsearch의 강력한 검색 기능을 **구조화된 JSON 객체 형태**로 표현할 수 있도록 설계되었으며,
단순한 키워드 검색부터 복잡한 조건 검색, 범위 검색, 집계(Aggregation)까지 모두 지원합니다.

즉, Query DSL은

> *Elasticsearch의 검색 엔진 기능을 JSON으로 명확하고 안전하게 표현하기 위한 전용 질의 언어*
> 라고 볼 수 있습니다.

---

## Query DSL의 특징

* JSON 기반 구조 → **가독성 및 유지보수성 우수**
* 모든 Elasticsearch 검색 기능 지원
* 조건 조합, 중첩, 필터링, 집계까지 확장 가능
* 애플리케이션 코드에서 동적 쿼리 생성에 적합

---

## Elasticsearch에서 쿼리를 사용하는 두 가지 방법

Elasticsearch에서 검색 쿼리를 작성하는 방식은 크게 **두 가지**가 있습니다.

### 1. 쿼리 스트링(Query String)

* REST API의 **URI 파라미터**에 쿼리를 직접 작성
* 예:

  ```
  /_search?q=status:ACTIVE AND age:[20 TO 30]
  ```

**장점**

* 간단한 조건 검색에 빠르고 편리
* 브라우저나 콘솔에서 즉석 테스트에 유용

**단점**

* 조건이 복잡해질수록 괄호 사용 증가
* 가독성 저하
* 문자열 기반이라 **문법 오류 발생 가능성 높음**

---

### 2. 쿼리 DSL (권장)

* REST API **요청 본문(body)** 에 JSON 형태로 쿼리를 작성
* Elasticsearch의 **모든 쿼리 스펙 지원**

```json
{
  "query": {
    "match": {
      "status": "ACTIVE"
    }
  }
}
```

**장점**

* 구조화된 JSON → 가독성 높음
* 복잡한 논리 조건 표현에 적합
* 프로그램 코드에서 안전하게 조합 가능
* 실무에서는 대부분 Query DSL 사용

**실제 서비스 환경에서는 Query DSL이 사실상 표준**

---

## Query DSL의 기본 구조

Query DSL은 크게 다음과 같은 구조를 가집니다.

```json
{
  "query": {
    ...
  }
}
```

이 안에 다양한 쿼리 타입(leaf / compound)을 조합해 검색 조건을 정의합니다.

---

## 리프 쿼리(Leaf Query) & 복합 쿼리(Compound Query)

Elasticsearch의 쿼리는 크게 **리프 쿼리**와 **복합 쿼리**로 나뉩니다.

---

### 1. 리프 쿼리 (Leaf Query)

* **단일 필드 또는 값에 대한 조건**을 정의
* 실제로 “무엇을 검색할지”를 담당
* 더 이상 하위 쿼리를 가지지 않음

대표 예시:

* `match`
* `term`
* `range`
* `exists`

```json
{
  "term": {
    "status": "ACTIVE"
  }
}
```

“이 조건이 참인가?”를 판단하는 **가장 작은 쿼리 단위**

---

### 2. 복합 쿼리 (Compound Query)

* 여러 개의 쿼리를 **논리적으로 조합**
* 리프 쿼리 또는 다른 복합 쿼리를 포함할 수 있음
* 검색 조건의 구조를 만드는 역할

대표 예시:

* `bool`
* `must`
* `should`
* `filter`
* `must_not`

```json
{
  "bool": {
    "must": [
      { "term": { "status": "ACTIVE" } }
    ],
    "filter": [
      { "range": { "age": { "gte": 20 } } }
    ]
  }
}
```

“어떤 조건들을 어떻게 조합할 것인가”를 담당

---

## 정리하면

* **리프 쿼리**: 실제 조건 (값 비교)
* **복합 쿼리**: 조건들의 논리 구조

둘을 조합해서 Elasticsearch의 강력한 검색 로직을 구성합니다.

---

## 한 줄 요약

> **Query DSL은 Elasticsearch의 검색 기능을 JSON으로 구조화해 표현하는 전용 질의 언어이며,
> 실무에서는 쿼리 스트링보다 Query DSL을 사용해 리프 쿼리와 복합 쿼리를 조합하는 방식이 표준이다.**

---

다음 단계로 이어가기 좋습니다:

* `match` vs `term` 차이
* `filter`와 `must` 성능 차이
* 검색(Query)과 필터(Filter)의 점수 계산 여부
* Elasticsearch Query DSL을 QueryDSL(JPA)과 비교 설명

원하시면 그중 하나 바로 이어서 정리해드릴게요.
