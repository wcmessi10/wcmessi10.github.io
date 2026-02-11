---
layout: post
title: Elasticsearch의 GeoPoint 필드 타입
tags: [Elasticsearch, GeoPoint]
comments: true
mathjax: true
author: 이희두
---

# 목적

지도 화면 기능에서 사용되는  
Elasticsearch의 **GeoPoint 필드 타입**을 정리한다.

---

# GeoPoint란?

GeoPoint는 Elasticsearch에서  
**특정 지리적 위치(한 점)**를 표현하기 위한 필드 타입이다.

- 위도(latitude)와 경도(longitude) 두 개의 실수 값으로 구성
- 지도 위의 한 점을 나타냄
- 다양한 입력 형식 지원
- 지리 기반 쿼리에서 핵심적으로 사용됨

---

## GeoPoint 입력 형식

GeoPoint는 여러 형식으로 입력 가능하다.

### 1. Object 형식

```json
"location": {
  "lat": 41.12,
  "lon": -71.34
}
````

---

### 2. 배열 형식

```json
"location": [-71.34, 41.12]
```

(순서: **lon, lat**)

---

### 3. 문자열 형식

```json
"location": "41.12,-71.34"
```

(순서: **lat,lon**)

---

### 4. Geohash 형식

```json
"location": "drm3btev3e86"
```

Geohash는 지도를 격자로 나누고
문자열로 좌표를 표현하는 방식이다.

---

# GeoPoint 활용 쿼리

GeoPoint는 다음과 같은 지리 쿼리에서 사용된다.

* Geo-bounding box query
* Geo-distance query
* Geo-shape query

---

# GeoPoint 매핑 설정

GeoPoint는 반드시 매핑에서 명시해야 한다.

```json
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```

---

# Spring Boot에서의 사용 예시

### 매핑 JSON 예시

```json
{
  "properties": {
    "areaId": { "type": "integer" },
    "areaName": { "type": "text" },
    "countryCode": { "type": "keyword" },
    "languageCode": { "type": "keyword" },
    "location": { "type": "geo_point" }
  }
}
```

---

### Document 클래스 예시

```java
@Getter
@Setter
@ToString
@Document(indexName = "<index_name>")
@Mapping(mappingPath = "<mapping_file_path>")
public class LocationDocument {

    @Id
    private Integer areaId;

    private String areaName;
    private String countryCode;
    private String languageCode;
    private GeoPoint location;
}
```

---

# Kibana에서 보이는 GeoPoint 예시

```json
"location": {
  "type": "Point",
  "coordinates": [127.095658, 37.325707]
}
```

GeoPoint는 내부적으로
GeoJSON Point 형태로 표현된다.

---

# 실무에서 느낀 포인트

* 좌표 순서(lon/lat) 실수 매우 빈번
* 문자열 vs 배열 형식 혼용 시 혼란 발생 가능
* 매핑 선언 누락 시 일반 숫자 필드로 저장되어 지리 쿼리 불가

---

# 참고

* [https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-point.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-point.html)
* [https://esbook.kimjmin.net/07-settings-and-mappings/7.2-mappings/7.2.6-geo](https://esbook.kimjmin.net/07-settings-and-mappings/7.2-mappings/7.2.6-geo)

```

