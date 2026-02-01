---
layout: post
title: OCR
tags: [Elasticsearch, Health Check, indicies, Sharding]
comments: true
mathjax: true
author: 이희두
---

# `/_cat/indices` 결과 해석 정리

`/_cat/indices`는
클러스터에 존재하는 인덱스 상태를 한눈에 보여주는 API입니다.

운영 환경에서:

* 인덱스 상태 점검
* 샤드 배치 확인
* 용량 모니터링

할 때 자주 사용됩니다.

---

## 예시 출력

```
health status index           uuid                     pri rep docs.count docs.deleted store.size pri.store.size
yellow open   my-index-000001 ...                      1   1   1200       0            88.1kb     88.1kb
green  open   my-index-000002 ...                      1   0   0          0            260b       260b
```

---

# 각 컬럼 설명

## health

인덱스의 샤드 할당 상태를 나타냅니다.

### green

* 모든 primary + replica 샤드가 정상 할당됨
* 가장 이상적인 상태

### yellow

* primary는 정상
* 일부 replica가 미할당
* 단일 노드 클러스터에서 replica=1이면 항상 yellow

운영에서 가장 흔한 원인:

* 노드 수 부족
* 디스크 부족
* 노드 장애

### red

* primary shard 일부가 미할당
* 데이터 접근 불가능 가능성 있음
* 즉시 대응 필요

---

## status

인덱스의 열림/닫힘 상태

### open

* 검색 가능
* 색인 가능

### closed

* 검색/색인 불가능
* 메타데이터만 유지됨
* 디스크 공간은 그대로 사용

실무 용도:

* 장기 보관 데이터 임시 비활성화
* 리소스 절약

---

## index

인덱스 이름

보통:

* 로그 인덱스는 날짜 단위
* rollover 정책 사용

예:

```
logs-2025.01.01
logs-2025.01.02
```

---

## uuid

인덱스의 내부 고유 ID

운영에서는 거의 볼 일 없음
주로 내부 관리용

---

## pri (Primary shards)

Primary shard 개수

중요 포인트:

* 인덱스 생성 시 결정
* 이후 변경 불가 (reindex 필요)

샤드 수가 많으면:

* 병렬 처리 증가
* 관리 비용 증가

---

## rep (Replica shards)

Replica shard 개수

역할:

* 장애 대비
* 검색 성능 향상

공식:

```
총 샤드 수 = pri × (1 + rep)
```

예:

```
pri=1, rep=1 → 총 2개 샤드
```

---

## docs.count

현재 인덱스에 살아있는 문서 수

주의:

* 실제 insert 횟수와 다를 수 있음
* update 시 내부적으로 delete+insert 처리

---

## docs.deleted

삭제된 문서 수 (논리 삭제)

중요 개념:

Elasticsearch는 즉시 물리 삭제 안 함
→ 나중에 merge 시 제거

값이 크면:

* 디스크 낭비 발생
* merge 필요

---

## store.size

Primary + Replica 전체 크기

실제 디스크 사용량에 가까움

예:

```
pri=1, rep=1
pri.store.size = 10GB
store.size = 20GB
```

---

## pri.store.size

Primary shard만의 크기

데이터 "원본" 크기 판단 시 기준

---

# 실무에서 꼭 알아야 할 포인트

## 1) 단일 노드에서 yellow는 정상

개발 서버에서:

```
rep=1
노드=1
```

이면 항상 yellow

해결 방법:

```
replica=0 설정
```

---

## 2) docs.deleted 많으면 성능 저하

해결:

```
forcemerge
```

하지만 운영에서는 신중하게 사용
(I/O 많이 사용)

---

## 3) 샤드 개수 설계 중요

샤드 너무 많으면:

* 메모리 낭비
* 클러스터 부담 증가

권장:

```
샤드당 10~50GB
```

---

## 4) 용량 모니터링 필수

store.size 기준으로:

* 노드 디스크 용량 확인
* watermark 초과 시 shard 이동 발생

---

# 운영 체크리스트

정상 클러스터:

* health = green
* docs.deleted 과다 아님
* shard 과다 아님
* store.size 모니터링 중


