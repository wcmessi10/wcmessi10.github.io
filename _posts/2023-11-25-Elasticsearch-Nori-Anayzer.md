---
layout: post
title: Elasticsearch에서 Nori 한글 형태소 분석기
tags: [Elasticsearch, 한글 형태소 분석기]
comments: true
mathjax: true
author: 이희두
---

## Elasticsearch에서 한글 형태소 분석기란?

Elasticsearch에서 **한국어 문서를 효과적으로 검색**하려면 반드시 **한글 형태소 분석기**가 필요합니다.

언어적 특성상:

* 영어 → **굴절어**
* 중국어 → **고립어**
* **한국어 → 교착어**

한국어는 **조사, 어미, 접미사, 접두사**가 결합되는 구조를 가지기 때문에,
단순한 공백 기반 토큰화 방식으로는 **정확한 검색 품질을 얻기 어렵습니다.**

### 한국어 검색이 어려운 이유

예:

```
먹다 → 먹었다 → 먹어본다 → 먹었겠지 → 먹고있다
```

의미는 동일하지만 형태가 계속 바뀌며,

```
사과 → 사과를 → 사과만 → 사과까지 → 사과라도
```

처럼 조사가 붙어 **형태 변화가 매우 다양**합니다.

따라서 Elasticsearch에서는
**한국어 문장 구조를 이해하고 토큰화하는 형태소 분석기가 필수**입니다.

---

## Elasticsearch에서 사용 가능한 오픈소스 한글 형태소 분석기

과거 Elasticsearch에서는 여러 외부 한글 분석기 플러그인을 사용했습니다.

대표적인 분석기:

* **아리랑(Arirang)**
* **은전한닢**
* **Open Korean Text (구 Twitter Korean Text)**

하지만:

* 유지보수 문제
* 성능
* Elasticsearch 버전 호환성
  등의 이슈로 **공식 지원 분석기의 필요성**이 커졌습니다.

---

## Nori란?

**Nori**는
Elasticsearch **6.6 버전부터 Elastic에서 공식적으로 지원하는 한글 형태소 분석기**입니다.

* Lucene 기반으로 개발
* Elasticsearch 기본 플러그인으로 제공
* **은전한닢의 mecab-ko-dic 사전을 재가공**하여 사용

### 핵심 특징

* 공식 지원 → **버전 호환성 & 안정성 보장**
* 빠른 분석 성능
* 대규모 인덱싱 및 검색에 적합
* Lucene 내부 구조와 깊게 통합

**현재 Elasticsearch에서 사실상 표준 한글 형태소 분석기**

---

## Nori 구조 및 동작 방식

Nori는 기본적으로 **형태소 분석 → 토큰 분리 → 필터 적용 → 검색 인덱싱** 구조로 동작합니다.

### 기본 분석 흐름

```
문장 입력
   ↓
형태소 분석
   ↓
토큰 분리
   ↓
불용어 제거 / 사용자 사전 적용
   ↓
검색용 토큰 생성
```

---

## Nori의 주요 구성 요소

### 1. Tokenizer — nori_tokenizer

* 한국어 문장을 **형태소 단위로 분해**
* Mecab 기반 알고리즘 사용
* 조사, 어미, 접사 자동 분리

---

### 2. Token Filter

| 필터                  | 설명       |
| ------------------- | -------- |
| nori_part_of_speech | 특정 품사 제거 |
| nori_readingform    | 발음 기반 검색 |
| stop                | 불용어 제거   |
| lowercase           | 소문자 변환   |

---

### 3. 사용자 사전 (User Dictionary)

* 도메인 특화 용어 등록 가능
* 서비스명, 브랜드명, 고유명사, 전문용어 처리 가능

예:

```
카카오뱅크
삼성페이
엘라스틱서치
```

실무에서는 **사용자 사전 설정이 검색 품질의 핵심**

---

## Nori 사용 예제

### Analyzer 설정

```json
PUT nori_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "korean_analyzer": {
          "type": "custom",
          "tokenizer": "nori_tokenizer"
        }
      }
    }
  }
}
```

---

### 형태소 분석 테스트

```json
POST nori_index/_analyze
{
  "analyzer": "korean_analyzer",
  "text": "엘라스틱서치에서 한국어 검색을 구현합니다"
}
```

### 결과 예시

```
엘라스틱서치
한국어
검색
구현
```

---

## Nori 도입 효과

| 항목       | 적용 전 | 적용 후     |
| -------- | ---- | -------- |
| 검색 정확도   | 낮음   | 매우 높음    |
| 조사/어미 처리 | 불가   | 자동 처리    |
| 형태 변화 대응 | 불가능  | 가능       |
| 검색 품질    | 낮음   | 고급 검색 가능 |

---

## 실무 적용 시 핵심 포인트

### 1. 반드시 text + analyzer 조합 사용

```json
"content": {
  "type": "text",
  "analyzer": "korean_analyzer"
}
```

---

### 2. keyword 필드와 멀티필드 조합

```json
"content": {
  "type": "text",
  "analyzer": "korean_analyzer",
  "fields": {
    "keyword": {
      "type": "keyword"
    }
  }
}
```

**검색 + 정렬 + 필터링을 동시에 만족**

---

### 3. 사용자 사전 적극 활용

* 도메인 특화 검색 품질 향상
* 서비스명, 기술명, 약어 필수 등록

---

## 한 줄 요약

> **Nori는 Elasticsearch에서 공식 지원하는 표준 한글 형태소 분석기로,
> 한국어 검색 품질을 획기적으로 개선해주는 핵심 컴포넌트이다.**


