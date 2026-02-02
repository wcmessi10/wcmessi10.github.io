---
layout: post
title: Elasticsearch에서 NativeQuery 성능 테스트
tags: [NativeQuery, Elasticsearch]
comments: true
mathjax: true
author: 이희두
---

# 목적 
기존 개발자들이 친숙한 SQL(Structured Query Language)과 동일 레벨로 검색엔진 부문에서는 DSL(Domain Specific Language)이라는 접근 방식이 있다.

개발환경에서의 SQL과 DSL의 접근 방식은 구조화 매칭 방식의 ORM 접근 방식 또는 별도의 native query를 사용하여 쿼리 명세 그대로 작성 접근하는 방식으로 나뉜다.

SQL에서 위의 Native Query를 사용하여 복잡한 질의문을 처리하는 것과 동일선상에 검색엔진 영역에서도 Native Query를 사용하는 방식을 준비하고자 진행 하였다.

# Elasticsearch에서 NativeQuery란?
Elasticsearch Client Library의 쿼리 빌더를 사용한 쿼리 구현체다.
(NativeSearchQuery는 엘라스틱 서치에서 더이상 사용하지 않는 이전 구현체다.)
# 성능 테스트 과정
**1억개 랜덤으로 들어간 0부터 10억까지의 수의 집합(document)**에서, 임의의 범위 쿼리를 주어 그 범위 내의 평균값을 구하는 테스트이다.

테스트 통해 구하고자 하는 것은 결과값을 도출해내는데 걸린 시간이다.

테스트 방식은 2가지다. (2가지 결과값을 비교하는 것은 아니다.)

- 엘라스틱 서버에서 테스트하는 방식(ElasticsearchOperation)
- 자바 클라이언트 안에서 테스트하는 방식(ElasticsearchClient)
![](/assets/img/캡처.PNG)

왼쪽 주황색 박스가 엘라스틱 서치 서버에서 구현한 테스트 코드(하단)와 ElasticsearchOperation 클래스에서 검색 메소드(상단),

오른쪽 초록색 박스가 자바 클라이언트에서 구현한 테스트 코드(하단)와  ElasticsearchClient 클래스에서 검색 메소드(상단)

필자는 gt 190000000(1억 9천만), lt 430000000(4억 3천만)의 범위를 통해 테스트를 주었다.
# 성능 테스트 결과
![](/assets/img/캡처3.PNG)
Elasticsearch 서버에서 결과값을 도출해낸 시간(2582ms)

![](/assets/img/캡처4.PNG)
클라이언트 통해서 결과값을 도출해낸 시간(4134ms)
