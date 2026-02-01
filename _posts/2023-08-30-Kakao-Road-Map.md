---
layout: post
title: 카카오 모빌리티 자동차 길찾기 API
tags: [Kakao]
comments: true
mathjax: true
author: 이희두
---


# 자동차 길찾기 API

자동차 경로 탐색을 제공하는 API로,
출발지–목적지–경유지를 기반으로 최적 경로 및 상세 안내 정보를 반환합니다.

---

# 요청 파라미터

| 이름           | 타입      | 설명          | 형식                                                                            | 필수 | 예시                                  |
| ------------ | ------- | ----------- | ----------------------------------------------------------------------------- | -- | ----------------------------------- |
| origin       | String  | 출발지         | `X,Y` 또는 `X,Y,name=이름`                                                        | O  | `127.111202,37.394912,name=판교역`     |
| destination  | String  | 도착지         | `X,Y` 또는 `X,Y,name=이름`                                                        | O  | 위와 동일                               |
| waypoints    | String  | 경유지         | `X,Y` 또는 `X,Y,name=이름`을 `\|` 또는 `%7C`로 연결                                     | X  | `127.11,37.39,name=A\|127.12,37.38` |
| priority     | String  | 경로 우선순위     | `RECOMMEND`(기본), `TIME`, `DISTANCE`                                           | X  | TIME                                |
| avoid        | String  | 제한 옵션       | `roadevent`, `ferries`, `toll`, `motorway`, `schoolzone`, `uturn` 등을 `\|`로 연결 | X  | toll|motorway                       |
| alternatives | Boolean | 대안 경로 제공    | true / false(기본)                                                              | X  | true                                |
| road_details | Boolean | 상세 도로 정보 제공 | true / false(기본)                                                              | X  | true                                |
| car_type     | int     | 차종          | 1~7                                                                           | X  | 1                                   |
| car_fuel     | String  | 유종          | GASOLINE(기본), DIESEL, LPG                                                     | X  | DIESEL                              |
| car_hipass   | Boolean | 하이패스 여부     | true / false(기본)                                                              | X  | true                                |
| summary      | Boolean | 요약 정보만 제공   | true / false(기본)                                                              | X  | true                                |

---

# 응답 구조

## 최상위 객체

| 필드       | 타입       | 설명       |
| -------- | -------- | -------- |
| trans_id | String   | 경로 요청 ID |
| routes   | Object[] | 경로 정보 배열 |

---

## routes 객체

| 필드          | 타입       | 설명        |
| ----------- | -------- | --------- |
| result_code | Int      | 결과 코드     |
| result_msg  | String   | 결과 메시지    |
| summary     | Object   | 경로 요약 정보  |
| sections    | Object[] | 구간별 경로 정보 |

---

# summary 객체

| 필드          | 타입       | 설명        |
| ----------- | -------- | --------- |
| origin      | Object   | 출발지 정보    |
| destination | Object   | 목적지 정보    |
| waypoints   | Object[] | 경유지 정보    |
| priority    | String   | 탐색 우선순위   |
| bound       | Object   | 바운딩 박스    |
| fare        | Object   | 요금 정보     |
| distance    | Int      | 총 거리(m)   |
| duration    | Int      | 총 소요시간(초) |

---

## 좌표 객체 (origin/destination/waypoint)

| 필드   | 타입     | 설명  |
| ---- | ------ | --- |
| name | String | 위치명 |
| x    | Double | 경도  |
| y    | Double | 위도  |

---

## bound (Bounding Box)

| 필드            | 설명     |
| ------------- | ------ |
| min_x / min_y | 좌하단 좌표 |
| max_x / max_y | 우상단 좌표 |

---

## fare

| 필드   | 설명       |
| ---- | -------- |
| taxi | 택시 요금(원) |
| toll | 통행 요금(원) |

---

# sections 객체

경유지가 있을 경우
**경유지 수 + 1 개의 섹션 생성**

예:

* section1: 출발지 → 경유지1
* section2: 경유지1 → 경유지2
* section3: 경유지2 → 목적지

| 필드       | 타입       | 설명                        |
| -------- | -------- | ------------------------- |
| distance | Int      | 구간 거리(m)                  |
| duration | Int      | 구간 시간(초)                  |
| bound    | Object   | 바운딩 박스 (summary=false일 때) |
| roads    | Object[] | 도로 정보                     |
| guides   | Object[] | 안내 정보                     |

---

# roads 객체 (summary=false일 때 제공)

| 필드            | 설명                     |
| ------------- | ---------------------- |
| name          | 도로명                    |
| distance      | 거리(m)                  |
| duration      | 시간(초)                  |
| traffic_speed | 교통 속도(km/h)            |
| traffic_state | 교통 상태                  |
| vertexes      | 좌표 배열 [x1,y1,x2,y2...] |

---

# guides 객체 (summary=false일 때 제공)

| 필드         | 설명          |
| ---------- | ----------- |
| name       | 지점명         |
| x / y      | 좌표          |
| distance   | 이전 지점 대비 거리 |
| duration   | 이전 지점 대비 시간 |
| type       | 안내 타입       |
| guidance   | 안내 문구       |
| road_index | 도로 인덱스      |

---

# summary=true일 때 응답 특징

* 도로 상세 정보 생략
* guides/roads 제외
* 요약 중심 정보만 제공
* 응답 크기 감소 → 모바일/실시간 처리에 유리

---

# 활용 팁 (실무 관점)

### 트래픽/비용 최적화

* 실시간 네비게이션 필요 없으면 `summary=true` 추천
* 서버–모바일 통신 비용 감소

### 실제 네비게이션 구현 시

* `summary=false` + guides 사용
* vertexes 기반 지도 Polyline 그리기 가능

### 경로 품질 제어

* 배달/모빌리티 서비스 → `priority=TIME`
* 연료 절감 → `DISTANCE`

