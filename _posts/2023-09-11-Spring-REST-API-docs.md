---
layout: post
title: Spring REST API Docs
tags: [API Documents, Spring REST API Docs]
comments: true
mathjax: true
author: 이희두
---

# Spring REST Docs 정리

## Spring REST Docs란

Spring REST Docs는
Spring MVC 기반 REST API를 테스트 코드 기반으로 문서화하는 도구입니다.

특징:

* 테스트 실행 시 실제 요청/응답을 기반으로 문서 생성
* 스펙과 코드 불일치 방지
* AsciiDoc + Asciidoctor로 문서 생성

즉,

“코드와 동기화되는 신뢰 가능한 API 문서”

를 만드는 데 목적이 있습니다.

---

# API 문서화란

클라이언트가 API를 사용하기 위해 필요한 정보를 정리하는 작업입니다.

포함 정보:

* 요청 URI
* HTTP Method
* Request Body
* Query Parameter
* Header
* Response Body
* 에러 코드

---

## API 문서의 쓰임새

### 개발자 지원

API 이해와 사용을 쉽게 해줌

### 통신 규약 정의

요청/응답 형식, 인증 방식 명시

### 오류 처리와 디버깅

에러 유형과 의미 제공

### 버전 관리

변경 사항 추적 가능

### 테스트와 통합

다른 서비스와 연동 용이

### 보안

인증/인가 방식 명확화

---

# Asciidoctor와 AsciiDoc

## AsciiDoc

문서 작성을 위한 마크업 언어

특징:

* 사람이 읽기 쉬움
* 작성 간편
* 구조화된 문서 작성 가능

---

## Asciidoctor

AsciiDoc 문서를 변환하는 도구

변환 가능 형식:

* HTML
* PDF
* EPUB

Spring REST Docs는
AsciiDoc → Asciidoctor → 최종 문서
흐름으로 동작합니다.

---

# 문서 생성 구조

일반적인 Gradle 기반 흐름:

1. 테스트 실행
2. snippets 생성
3. Asciidoctor로 변환
4. HTML 생성
5. bootJar에 포함 또는 정적 리소스로 복사

---

## Snippet이란

Snippet은

“문서 조각”

입니다.

예:

* request-fields
* response-fields
* path-parameters
* query-parameters
* http-request
* http-response

테스트 1개당 여러 snippet이 생성됩니다.

이 snippet들을 조합해 최종 문서를 만듭니다.

---

# Spring REST Docs의 장점

### 스펙 불일치 방지

Request/Response가 실제 코드와 다르면 테스트 실패

→ 문서 신뢰도 매우 높음

---

### 코드 기반 문서화

수기 문서 관리 필요 없음

---

### 운영 환경에 적합

정확성이 중요할 때 유리

---

# Spring REST Docs의 단점

### 테스트 작성 필수

테스트 코드 없으면 문서 생성 불가

---

### 변경 대응 비용

API 변경 시 테스트도 수정 필요

빠른 프로토타이핑 단계에서는 부담 가능

이 경우:
Swagger/OpenAPI가 더 편리할 수 있음

---

# API 문서 생성 흐름

```
테스트 코드 작성
→ test 실행
→ snippet 생성
→ .adoc 문서 생성
→ HTML 변환
```

---

# 테스트 코드 핵심 요소

## @WebMvcTest

컨트롤러 중심 테스트 로딩

---

## @AutoConfigureRestDocs

REST Docs 자동 설정 활성화

---

## @Test

테스트 메소드 지정

---

## MockMvc

서버 실행 없이 MVC 테스트 가능

### 주요 메소드

perform()
요청 수행

andExpect()
응답 검증

andDo()
문서화 등 후처리 수행

---

# 템플릿 작성

경로:

```
src/docs/asciidoc/*.adoc
```

AsciiDoc 문법 사용

---

## include 기능

여러 snippet 또는 adoc 파일을 조합 가능

예:

```
include::{snippets}/user-create/http-request.adoc[]
```

---

# build 과정

1. MockMvc.andDo(document()) 실행
2. snippets 생성
3. asciidoctor task 실행
4. HTML 생성
5. 정적 리소스 포함

---

# 언제 쓰는 게 좋은가

### Spring REST Docs 추천 상황

* 금융/보안 서비스
* 외부 API 공개
* 문서 정확도 중요
* 운영 안정성 중요

---

### Swagger 추천 상황

* 빠른 개발
* API 변경 잦음
* 프론트 협업용 즉시 문서 필요

---

# 한 줄 정리

Spring REST Docs는 테스트 기반으로 실제 요청/응답을 검증하며 생성되는, 신뢰성 높은 API 문서화 도구이다.
