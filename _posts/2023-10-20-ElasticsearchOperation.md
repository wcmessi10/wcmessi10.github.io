---
layout: post
title: Spring Data Elasticsearch
tags: [Elasticsearch, Spring Data Elasticsearch, Elasticsearch Client, ElasticsearchOperation]
comments: true
mathjax: true
author: 이희두
---

# ElasticsearchOperations 정리 (Spring Data Elasticsearch)

## 1) ElasticsearchOperations 역할

* `ElasticsearchOperations`는 **Spring Data Elasticsearch의 상위 레벨 API**로, `Query`(또는 `NativeQuery` 등)를 받아 **Elasticsearch에 요청을 수행**하는 진입점입니다.
* Spring Data의 매핑/컨버팅(엔티티 ↔ Document), 인덱스 메타데이터 처리 등 **스프링 친화적인 기능**을 포함합니다.

즉, “클라이언트 직접 호출”보다 “스프링 데이터 방식으로 CRUD/검색을 편하게” 하는 인터페이스입니다. 

---

## 2) 내부적으로 무엇을 감싸는가 (중요한 최신 포인트)

예전에는 `ElasticsearchRestTemplate`이 **Elasticsearch의 High Level REST Client(HLRC)** 기반 구현이었습니다. 

하지만 현재(5.0 이후) Spring Data Elasticsearch는 기본적으로 **Elasticsearch의 새 Java API Client (`ElasticsearchClient`)** 기반을 사용하고, **기존 `RestHighLevelClient` 기반 코드는 deprecated 처리**되었습니다.

정리하면:

* HLRC(High Level REST Client): Elasticsearch 측에서 deprecated (7.15부터)
* Spring Data Elasticsearch: 5.0부터 새 `ElasticsearchClient` 기반이 기본, HLRC 기반 코드는 deprecated 및 패키지 분리

따라서 “ElasticsearchOperations나 HLRC 둘 중

* 신규/권장: `ElasticsearchOperations`(내부적으로 새 `ElasticsearchClient` 사용)
* 레거시/비권장: HLRC 직접 사용 또는 HLRC 기반 template

---

## 3) Operations 구성(인터페이스 관계)

Spring Data Elasticsearch는 관심사를 분리한 operations를 제공합니다.

* `IndexOperations`

  * 인덱스 생성/삭제/매핑/alias 등 “인덱스 레벨” 작업

* `DocumentOperations`

  * id 기반 저장/업데이트/조회 등 “단건 문서” 중심 작업

* `SearchOperations`

  * query 기반 검색/집계 등 “검색” 중심 작업

* `ElasticsearchOperations`

  * 실무에서 주로 쓰는 상위 인터페이스로, 문서 작업과 검색 작업을 포함하는 형태로 제공됩니다(버전에 따라 노출되는 메서드/구성은 조금씩 다를 수 있지만, 목적은 “CRUD + Search 편의”입니다).

---

## 4) ElasticsearchRestTemplate / ElasticsearchTemplate 관련

* (과거 문서 기준) `ElasticsearchRestTemplate`은 `ElasticsearchOperations`의 구현체이며, HLRC 기반으로 동작한다고 명시되어 있었습니다.
* `ElasticsearchTemplate`은 과거 구현체로, 오래전부터 대체 방향이었고(버전대에 따라 표현 차이 존재) 현재는 “새 클라이언트 기반 구성”으로 넘어온 상태입니다.

---

## 5) 스프링 빈 이름과 주입 팁(버전 5.0+ 기준)

Spring Data Elasticsearch 5.0의 마이그레이션 가이드에 따르면, 새 클라이언트 구성 시 다음이 제공됩니다:

* `RestClient`(low-level)
* `ElasticsearchClient`(new Java API client)
* `ElasticsearchOperations` 빈: 이름이 `elasticsearchOperations`, `elasticsearchTemplate` 로 제공

따라서 보통은

* 타입으로 `ElasticsearchOperations`를 주입받으면 충분하고
* 이름으로 구분해야 하는 상황이면 위 빈 이름을 참고하면 됩니다.

---

## 6) 한 줄 결론(추천 방향)

* 실무에서는 가능한 한 **`ElasticsearchOperations`(또는 Repository)** 중심으로 사용하고,
* 클라이언트 직접 호출은 **deprecated 흐름**이므로 신규 개발에서는 피하는 게 좋습니다.

## 7) 출처
([Spring Data Elasticsearch 버전 관련 공식 문서][1])
([Spring Data Elasticsearch 참고 공식 문서][2])
([Java Elasticsearch REST Client][3])

[1]: https://docs.spring.io/spring-data/elasticsearch/reference/migration-guides/migration-guide-4.4-5.0.html "Upgrading from 4.4.x to 5.0.x :: Spring Data Elasticsearch"
[2]: https://docs.spring.io/spring-data/elasticsearch/docs/4.1.16-SNAPSHOT/reference/html "Spring Data Elasticsearch - Reference Documentation"
[3]: https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high.html "Java High Level REST Client"
