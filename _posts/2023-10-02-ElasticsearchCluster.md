---
layout: post
title: Elasticsearch Cluster
tags: [Elasticsearch Cluster, Sharding, Replication]
comments: true
mathjax: true
author: 이희두
---

# Elasticsearch Cluster 정리

## Elasticsearch Cluster란

Elasticsearch Cluster는
여러 노드(Node)가 네트워크로 연결되어 하나의 시스템처럼 동작하는 Elasticsearch 인스턴스들의 집합입니다.

클러스터는 다음을 제공합니다:

* 데이터 분산 저장
* 수평 확장(Scale-out)
* 고가용성(High Availability)
* 장애 복구(Replica 기반)

Elasticsearch는 기본적으로 **항상 클러스터 모드로 동작**합니다.
노드가 1개뿐이어도 “단일 노드 클러스터”입니다.

---

# Cluster

* 최소 1개 이상의 노드로 구성
* 하나의 클러스터는 하나의 논리적 데이터 저장소
* 다른 클러스터와는 기본적으로 데이터 공유 불가

(단, Cross-Cluster Search/Replication으로 예외적으로 가능)

---

# Node

Node는 Elasticsearch 프로세스 1개입니다.
즉, ES가 실행 중인 인스턴스 1개 = Node 1개입니다.

### Node 역할 (중요)

최신 ES에서는 노드 역할을 분리합니다:

### Master-eligible node

* 클러스터 상태 관리
* 샤드 배치 결정
* 메타데이터 관리

### Data node

* 실제 데이터 저장
* 검색/집계 처리

### Coordinating node

* 클라이언트 요청을 받아 분산 처리 후 결과 취합

### Ingest node

* 색인 전 파이프라인 처리

실무에서는:

* 소규모: 역할 혼합
* 대규모: 역할 분리

---

# Index

Index는 문서(Document)의 논리적 집합입니다.
RDB의 “테이블”과 유사합니다.

### 구성 구조

```
Index
 ├── Shard (Primary/Replica)
 │     ├── Document
 │     ├── Document
```

* 인덱스는 여러 샤드로 분할됨
* 샤드는 여러 노드에 분산 저장됨

---

### RDB vs Elasticsearch 모델링 차이

RDB:

* 정규화
* Join 중심

Elasticsearch:

* 비정규화
* Document 중심 설계
* Join 거의 사용하지 않음

실무 원칙:

> 조회 패턴에 맞춰 문서를 설계한다.

---

# 네트워크 포트

### HTTP 포트

* 기본 9200
* REST API 통신
* 클라이언트 접속

### Transport 포트

* 기본 9300
* 노드 간 통신
* 샤드 복제/분산 처리

---

# 노드 배치 권장사항

### 권장

* 물리 서버 1대 = 노드 1개
* 리소스 충돌 방지
* 안정적인 GC 관리

### 가능

* 한 서버에 여러 노드 실행
* 개발/테스트 환경에서만 추천

---

# cluster.name 중요성

같은 클러스터로 묶이려면:

```
cluster.name = 동일
```

이어야 합니다.

다르면:

* 완전히 다른 클러스터
* 서로 인식 못함

---

### 실무에서 더 중요한 설정

cluster.name보다 더 중요한 건:

```
cluster.initial_master_nodes
discovery.seed_hosts
```

이 설정이 없으면 클러스터 형성 실패 가능.

---

# Spring Data Elasticsearch와 클러스터

Spring Data Elasticsearch는:

* 단일 노드
* 다중 노드 클러스터

모두 연결 가능

Spring 애플리케이션은 보통:

* ElasticsearchOperations
* ElasticsearchRepository

를 통해 상위 레벨로 사용합니다.

애플리케이션은 클러스터 구조를 몰라도 되며,
클러스터 내부에서 자동으로 분산 처리됩니다.

---

# 실무 핵심 포인트

## 1) 최소 운영 구성

운영 환경 추천:

* Master node 3개 (quorum)
* Data node 2~3개 이상

이유:

* split brain 방지
* 장애 대비

---

## 2) Replica 필수

Replica 없으면:

* 장애 시 데이터 손실 가능
* 검색 부하 분산 불가

---

## 3) 샤드 과다 금지

샤드 많으면:

* 메모리 낭비
* GC 증가
* 성능 저하

권장:

* 샤드당 10~50GB

---

## 4) 디스크 watermark 중요

디스크 부족 시:

* shard 이동 발생
* cluster yellow/red 가능

---

# 한 줄 요약

Elasticsearch Cluster는 여러 노드가 협력해 데이터를 분산 저장하고 처리하는 구조로, 확장성과 고가용성을 핵심 목표로 설계된 분산 검색/분석 시스템이다.


