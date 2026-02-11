---
layout: post
title: Elasticsearch의 GeoShape 쿼리를 이용한 원형 지리 테스트
tags: [Elasticsearch, GeoSahpe Query, Spring Boot]
comments: true
mathjax: true
author: 이희두
---

# 목적

지도 API에서 활용하기 위해  
Elasticsearch의 GeoShape 쿼리를 이용해 **원형 범위 내 좌표를 검색하는 테스트**를 진행했다.

---

# Elasticsearch의 GeoShape 쿼리란?

GeoShape 쿼리는  
쿼리에 지리적 도형(GeoJSON)을 조건으로 추가하여  
`geo_point` 또는 `geo_shape` 데이터를 필터링하는 기능이다.

즉, **"특정 지리적 영역에 속하는 데이터만 조회"**하는 쿼리라고 볼 수 있다.

---

## 기본 GeoShape 쿼리 예시

```json
GET /<index_name>/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_shape": {
          "location": {
            "shape": {
              "type": "envelope",
              "coordinates": [[13.0, 53.0], [14.0, 52.0]]
            },
            "relation": "intersects"
          }
        }
      }
    }
  }
}
````

---

## 예시 응답

```json
{
  "hits": {
    "total": { "value": 1 },
    "hits": [
      {
        "_source": {
          "name": "Example Place",
          "location": [13.400544, 52.530286]
        }
      }
    ]
  }
}
```

---

# 주요 개념

### shape

GeoJSON 형식으로 표현한 도형 데이터

### relation

도형과 문서 간의 공간 관계 조건

---

## relation 종류

| 옵션         | 설명                      |
| ---------- | ----------------------- |
| INTERSECTS | (기본값) 쿼리 도형과 교차하는 모든 문서 |
| DISJOINT   | 도형과 전혀 겹치지 않는 문서        |
| WITHIN     | 도형 내부에 완전히 포함된 문서       |
| CONTAINS   | 도형이 문서를 포함하는 경우         |

---

# GeoJSON으로 원형(Circle) 표현

```json
{
  "type": "circle",
  "radius": "40m",
  "coordinates": [30, 10]
}
```

* type: circle
* radius: 반지름 (단위 포함 문자열)
* coordinates: 중심점 (경도, 위도)

---

# Spring Boot에서 GeoShape 쿼리 사용 예시

```java
public List<LocationDocument> findByGeoPointRangeCircle(Map<String,Object> range){

    Map<String, Object> shapeMap = Map.of(
            "type", "circle",
            "coordinates", List.of(range.get("lon"), range.get("lat")),
            "radius", range.get("radius")
    );

    JsonData jsonData = JsonData.of(shapeMap);

    GeoShapeQuery geoShapeQuery = GeoShapeQuery.of(f -> f
            .field("location")
            .shape(s -> s
                    .relation(GeoShapeRelation.Within)
                    .shape(jsonData)
            ));

    NativeQuery query = NativeQuery.builder()
            .withQuery(q -> q.bool(b -> b.must(m -> m.geoShape(geoShapeQuery))))
            .withPageable(Pageable.ofSize(100))
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

반지름과 중심점을 기반으로
**원형 내부에 포함된 좌표 데이터만 조회**하는 코드이다.

---

# 테스트 결과

원형 영역 내의 좌표들이 정상적으로 검색되는 것을 확인했다.
GeoShape 쿼리를 활용하면 원, 사각형, 다각형 등 다양한 도형 기반 검색이 가능하다.

---

# 참고

* [https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-shape-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-shape-query.html)
* [https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest-circle-processor.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest-circle-processor.html)

```
