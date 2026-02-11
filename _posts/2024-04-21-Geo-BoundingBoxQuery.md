---
layout: post
title: Elasticsearch의 Geo-BoundingBoxQuery
tags: [Elasticsearch, Geo-BoundingBoxQuery]
comments: true
mathjax: true
author: 이희두
---

# 목적

지도 화면 기능에서 사용되는 **Geo Bounding Box Query(사각형 범위 검색)**를 정리한다.

---

# Geo Bounding Box Query란?

Geo Bounding Box Query는 **사각형 영역**을 기준으로 검색하는 지리 쿼리이다.

- `top_left`(좌측 상단)와 `bottom_right`(우측 하단) 좌표를 지정
- 해당 사각형 안에 포함되는 `geo_point` 필드를 가진 문서를 조회

---

# Geo Bounding Box Query 예시 (Elasticsearch)

```json
GET <index_name>/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.01,
              "lon": -71.12
            }
          }
        }
      }
    }
  }
}
````

---

# Spring Boot에서의 구현 예시

아래 예시는 `topLeft`, `bottomRight` 좌표를 받아 사각형 내부에 포함되는 문서를 조회한다.

```java
public List<LocationDocument> findByGeoPointRange(Map<String, Object> range) {

    Map<String, Double> topLeft = (Map<String, Double>) range.get("topLeft");
    Map<String, Double> bottomRight = (Map<String, Double>) range.get("bottomRight");

    GeoBounds geoBounds = new GeoBounds.Builder()
            .tlbr(tlbr -> tlbr
                    .topLeft(p -> p.latlon(ll -> ll
                            .lat(topLeft.get("lat"))
                            .lon(topLeft.get("lon"))))
                    .bottomRight(p -> p.latlon(ll -> ll
                            .lat(bottomRight.get("lat"))
                            .lon(bottomRight.get("lon")))))
            .build();

    NativeQuery query = NativeQuery.builder()
            .withQuery(q -> q.geoBoundingBox(v -> v
                    .field("location")
                    .boundingBox(geoBounds)))
            .build();

    SearchHits<LocationDocument> hits =
            elasticsearchOperations.search(query, LocationDocument.class);

    List<LocationDocument> result = new ArrayList<>();
    for (SearchHit<LocationDocument> hit : hits) {
        result.add(hit.getContent());
    }

    return result;
}
```

* `GeoBounds`에 `top_left`, `bottom_right`를 세팅
* `NativeQuery`에서 `geoBoundingBox` 필터로 검색
* 조회 결과(`SearchHits`)에서 Document만 추출하여 반환

---

# 테스트 결과

사각형 범위 내부에 포함되는 문서들이 정상적으로 조회되는 것을 확인했다.
Geo Bounding Box Query는 지도 화면에서 **현재 화면 영역(뷰포트) 기준 데이터 로딩**에 특히 유용하다.

---

# 참고

* [https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-bounding-box-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-bounding-box-query.html)
* [https://esbook.kimjmin.net/07-settings-and-mappings/7.2-mappings/7.2.6-geo](https://esbook.kimjmin.net/07-settings-and-mappings/7.2-mappings/7.2.6-geo)

