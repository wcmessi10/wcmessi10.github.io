---
layout: post
title: Leaf Query
tags: [Elasticsearch, QueryDSL, Leaf Query]
comments: true
mathjax: true
author: 이희두
---

## 리프 쿼리 (Leaf Query)

**리프 쿼리(Leaf Query)** 는
Elasticsearch에서 **특정 필드의 값과 직접 비교하여 조건을 판단하는 가장 기본적인 쿼리 단위**입니다.

* 단일 필드 기준으로 동작
* 더 이상 하위 쿼리를 포함하지 않음
* 실제로 “무엇을 찾을 것인가”를 담당

---

## 리프 쿼리의 주요 분류

리프 쿼리는 크게 다음 세 가지로 나뉩니다.

1. **전문 쿼리 (Full Text Query)**
2. **용어 수준 쿼리 (Term-level Query)**
3. **범위 쿼리 (Range Query)**

---

## 1. 전문 쿼리 (Full Text Query)

* **텍스트가 많은 필드**에서 특정 용어를 검색할 때 사용
* 인덱스 매핑 시 `text` 타입으로 매핑된 필드에서 사용 권장
* **분석기(Analyzer)** 를 통해 토큰화, 소문자 변환, 불용어 제거 등을 거친 뒤 검색 수행

“사람이 읽는 문장 검색”에 적합

---

### 1.1 match query

* 전체 텍스트 중 특정 용어를 검색
* 검색하려는 **필드명을 반드시 알고 있어야 함**

  * 필드를 모를 경우 `_mapping` 조회 필요
* 기본적으로 **공백은 OR 조건**

```json
{
  "match": {
    "name": "Lionel Messi"
  }
}
```

위 쿼리는 내부적으로:

* `lionel OR messi`

로 처리되므로
`Lionel Scaloni`, `Thiago Messi` 같은 문서도 결과에 포함될 수 있습니다.

#### AND 조건으로 검색하려면

```json
{
  "match": {
    "name": {
      "query": "Lionel Messi",
      "operator": "and"
    }
  }
}
```

✔ 일반적인 분석기는 대문자를 소문자로 변환함

---

### 1.2 match_phrase query

* **구(phrase)** 단위 검색
* 단어의 **순서까지 정확히 일치**해야 함
* `match`보다 더 엄격한 검색

```json
{
  "match_phrase": {
    "name": "Lionel Andres Messi"
  }
}
```

X `Messi Lionel`, `Lionel Messi Andres` → 검색되지 않음

---

### 1.3 multi-match query

* 검색어가 **어떤 필드에 들어있는지 모를 때 사용**
* 여러 필드를 대상으로 동시에 검색 가능
* 전문 검색이므로 `text` 타입 필드에서 사용 권장

```json
{
  "multi_match": {
    "query": "Messi",
    "fields": ["name", "description", "bio"]
  }
}
```

“검색창 하나로 여러 필드를 검색해야 할 때” 가장 많이 사용

---

### 1.4 query_string query

* 연산자를 기준으로 **쿼리 문자열을 직접 파싱**
* AND, OR, NOT, 괄호 등 논리 연산 지원
* 문법 오류에 민감 → 실무에서는 제한적으로 사용

```bash
curl "localhost:9200/sales-records/_search" -H "Content-Type:application/json" -d '
{
  "query": {
    "query_string": {
      "default_field": "country",
      "query": "(south africa) OR (south korea)"
    }
  }
}'
```

해석:

* `(south africa)` → `south AND africa`
* `(south korea)` → `south AND korea`

즉,

```
(south AND africa) OR (south AND korea)
```

---

## 2. 용어 수준 쿼리 (Term-level Query)

* **정확한 값 일치**를 기준으로 검색
* 분석기를 거치지 않음
* 숫자, 날짜, 범주형 데이터 검색에 적합
* 보통 `keyword`, 숫자, 날짜 타입 필드에서 사용

“기계적인 값 비교”에 적합

---

### 2.1 term query

* **반드시 `keyword` 타입으로 매핑된 필드에서 사용**
* 검색어가 **분석되지 않음**
* 대소문자 구분, 공백 포함 여부까지 정확히 일치해야 함

```json
{
  "term": {
    "name.keyword": "Lionel Messi"
  }
}
```

#### match vs term 차이

| 구분     | match query | term query |
| ------ | ----------- | ---------- |
| 분석기 사용 | O           | X          |
| 토큰화    | O           | X          |
| 대소문자   | 무시          | 구분         |
| 사용 필드  | text        | keyword    |

---

### 2.2 terms query

* 여러 개의 값을 한 번에 검색
* `IN` 조건과 유사
* `keyword` 타입 필드에서 사용

```json
{
  "terms": {
    "status": ["ACTIVE", "PENDING"]
  }
}
```

---

## 3. 범위 쿼리 (Range Query)

* **숫자, 날짜, IP** 값의 범위를 지정해 검색
* 문자열(`text`, `keyword`) 타입은 사용 불가
* 쿼리에 사용하는 날짜 포맷과 필드 포맷이 반드시 일치해야 함

```json
{
  "range": {
    "age": {
      "gte": 20,
      "lt": 30
    }
  }
}
```

### 지원 파라미터

| 파라미터 | 의미 |
| ---- | -- |
| gte  | 이상 |
| gt   | 초과 |
| lte  | 이하 |
| lt   | 미만 |

---

## 리프 쿼리 선택 기준 요약

* 문장 검색, 유사도 검색 → **match / match_phrase**
* 여러 필드 검색 → **multi-match**
* 정확한 값 비교 → **term / terms**
* 숫자·날짜 범위 → **range**

---

## 한 줄 요약

> **리프 쿼리는 Elasticsearch 검색의 최소 단위로,
> 전문 검색(text)에는 match 계열, 정확한 값 비교에는 term 계열,
> 범위 조건에는 range 쿼리를 사용하는 것이 핵심이다.**
