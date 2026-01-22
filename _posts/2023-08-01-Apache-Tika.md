---
layout: post
title: Apache Tika
tags: [Java, Text Extraction, Tika]
comments: true
mathjax: true
author: 이희두
---

## Apache Tika 개요

**Apache Tika**는 Apache Software Foundation에서 관리하는
**Java 기반 콘텐츠 감지(Content Detection) 및 분석 프레임워크**입니다.

수천 가지 이상의 파일 형식에서 **텍스트와 메타데이터를 자동으로 추출**할 수 있으며,
Java 라이브러리뿐만 아니라 **CLI(Command Line Interface)** 및 **서버 형태**로도 제공되어
다양한 언어·환경에서 활용할 수 있습니다.

---

## Apache Tika의 주요 기능

### 1. 텍스트 추출 (Text Extraction)

* PDF, Word, Excel, HTML, 이미지(OCR 연계) 등 다양한 문서에서 텍스트 추출
* 검색, 인덱싱, 로그 분석, 자연어 처리(NLP) 전처리에 활용 가능

### 2. 메타데이터 추출 (Metadata Extraction)

* 제목, 작성자, 생성일, 수정일, MIME 타입 등 문서 메타정보 추출
* 문서 분류, 감사 로그, 문서 관리 시스템(DMS)에 활용

### 3. 파일 형식 탐지 (File Type Detection)

* 확장자가 아닌 **파일 실제 내용 기반**으로 형식 탐지
* 위·변조 파일, 확장자 변경 파일 판별에 유용

### 4. 언어 감지 (Language Detection)

* 문서 본문의 언어 자동 식별
* 다국어 검색, 번역, 언어별 분석 파이프라인 구성에 활용

### 5. 대용량 처리 지원

* 스트리밍 기반 파싱
* 대용량 파일 및 대량 문서 처리에 적합

---

## 핵심 파서 및 처리 클래스 정리

### AutoDetectParser

* 입력 데이터의 **파일 형식을 자동 감지**
* 텍스트, PDF, Word, HTML, 이미지 등 형식에 맞는 파서를 내부적으로 선택
* 실무에서 가장 많이 사용되는 기본 파서

> “파일 타입을 신경 쓰지 않고 넣으면, 알아서 처리해주는 파서”

---

### HTMLParser

* HTML 문서를 파싱하여 **텍스트와 메타데이터 추출**
* 다양한 HTML 구조를 지원
* 크롤링 데이터, 웹 문서 분석 시 활용

---

### DefaultDetector

* 파일의 **MIME 타입을 감지**하는 역할
* 확장자 + 파일 시그니처 + 내부 구조를 종합적으로 분석
* 보안, 업로드 파일 검증 로직에 자주 사용

---

## ContentHandler 계열 클래스

### BodyContentHandler

* 파싱된 문서의 **본문 텍스트를 수집**
* XHTML `<body/>` 태그 내부의 내용만 처리
* `<body/>` 태그 자체는 전달되지 않음

> “문서의 실제 텍스트 내용만 필요할 때 가장 기본적으로 사용하는 핸들러”

---

### ToHTMLContentHandler

* SAX 이벤트를 받아 **HTML 문자열로 직렬화**
* 유효한 HTML 구조를 가진 문서만 처리 가능
* 문서를 HTML 형태로 변환해야 할 때 사용

---

### TeeContentHandler

* 하나의 파싱 결과를 **여러 ContentHandler로 동시에 전달**
* 예:

  * 하나는 텍스트 추출
  * 하나는 HTML 변환
  * 하나는 로그 저장

> “한 번의 파싱으로 여러 목적을 동시에 달성”

---

## 변환 및 파싱 컨텍스트

### TransformHandler

* 문서를 **다른 형식으로 변환**하는 데 사용
* 예:

  * PDF → HTML
  * 문서 텍스트 → 가공된 포맷
* `Transformer`와 함께 사용됨

---

### ParseContext

* 파싱 과정에서 필요한 **컨텍스트(설정 정보)**를 전달하는 객체
* 파서 동작 방식을 세밀하게 제어 가능

활용 예:

* 특정 파서 강제 지정
* 메타데이터 필터 설정
* OCR, 보안 설정, 커스텀 파서 연동

> “파서에게 ‘어떤 조건과 환경에서 파싱할지’를 알려주는 설정 객체”

---

## 한눈에 보는 역할 정리

| 구성요소                 | 역할                |
| -------------------- | ----------------- |
| AutoDetectParser     | 파일 형식 자동 감지 + 파싱  |
| DefaultDetector      | MIME 타입 판별        |
| HTMLParser           | HTML 문서 파싱        |
| BodyContentHandler   | 본문 텍스트 수집         |
| ToHTMLContentHandler | HTML 형태로 변환       |
| TeeContentHandler    | 여러 Handler로 동시 전달 |
| TransformHandler     | 문서 변환 처리          |
| ParseContext         | 파싱 환경/설정 전달       |

---

## 정리 한 줄 요약

> **Apache Tika는 “파일 형식을 몰라도 텍스트와 메타데이터를 자동으로 추출할 수 있게 해주는 범용 문서 분석 프레임워크”이며,
> AutoDetectParser + ContentHandler 조합이 실무 사용의 핵심이다.**

