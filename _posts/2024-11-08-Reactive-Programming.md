---
layout: post
title: 선언형 프로그래밍 (Declarative Programming)
tags: [Declarative Programming]
comments: true
mathjax: true
author: 이희두
---

선언형 프로그래밍은 **“어떻게(How)”보다 “무엇을(What)”**에 집중하는 방식이다.  
절차를 하나하나 기술하기보다, 목표 상태를 기술한다.

대표 예시:

- SQL
- React
- Stream API
- Reactive Programming

---

## 선언형 프로그래밍의 특징

### 가독성
코드가 간결해지고 의도가 명확해진다.

### 유지보수성
로직이 추상화되어 재사용성과 확장성이 높다.

### 병렬 처리 친화성
불변 데이터 + 함수형 스타일 덕분에 병렬 처리에 유리하다.

### 추상화 수준이 높음
구현 세부사항을 숨기고 핵심 로직에 집중할 수 있다.

---

# Java에서의 선언형 프로그래밍 예시

- Stream API
- Optional
- Reactive Streams

---

## 예시 코드

### 선언형 스타일

```java
Optional.ofNullable(data.get("searchCondition"))
        .map(v -> v.toString().split("\\s+"))
        .ifPresent(words ->
            Arrays.stream(words)
                .forEach(word ->
                    boolQueryBuilder.must(q ->
                        q.queryString(qs ->
                            qs.fields("name")
                              .query("*" + word + "*")))));
````

```java
List<Map<String,Object>> result =
    searchOperations.search(query, Document.class)
        .stream()
        .map(hit -> {
            Document doc = hit.getContent();
            Map<String,Object> map = new HashMap<>();
            map.put("id", doc.getId());
            map.put("name", doc.getName());
            return map;
        })
        .collect(Collectors.toList());
```

---

### 명령형 스타일

```java
if (data.containsKey("searchCondition")) {
    String[] words =
        data.get("searchCondition").toString().split("\\s+");

    for (String word : words) {
        boolQueryBuilder.must(q ->
            q.queryString(qs ->
                qs.fields("name")
                  .query("*" + word + "*")));
    }
}
```

```java
List<Map<String,Object>> list = new ArrayList<>();

for (SearchHit<Document> hit : hits) {
    Document doc = hit.getContent();

    Map<String,Object> map = new HashMap<>();
    map.put("id", doc.getId());
    map.put("name", doc.getName());

    list.add(map);
}
```

---

# 선언형 vs 명령형 성능 비교

선언형 프로그래밍은 추상화 계층이 높기 때문에
**이론적으로는 약간의 오버헤드가 발생할 수 있다.**

하지만 실제 성능 차이는 다음에 더 크게 좌우된다:

* 구현체 최적화 수준
* 데이터 규모
* 병렬 처리 여부
* I/O 비중

대부분의 실무 환경에서는
**성능보다 생산성과 유지보수성이 더 큰 이점**이 된다.

---

# Reactive Streams

Reactive Streams는
**비동기 스트림 처리를 위한 표준 API 및 프로토콜**이다.

특징:

* Non-blocking
* Backpressure 지원
* 대용량 데이터 처리에 적합

---

## Spring WebFlux

Spring WebFlux는
Reactive Streams 구현체인 **Project Reactor** 기반이다.

* Mono
* Flux
* 비동기 논블로킹 처리

---

# 결론

선언형 프로그래밍은

> 코드를 "작동 방식"이 아니라 "의도" 중심으로 작성하게 만든다.

성능이 중요한 구간에서는 명령형,
일반 로직에서는 선언형을 사용하는 **균형 잡힌 접근**이 가장 현실적이다.

