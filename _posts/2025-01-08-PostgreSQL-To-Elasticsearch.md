---
layout: post
title: PostgreSQL to Elasticsearch Migration 과정
tags: [PostgreSQL, ELK, Elasticsearch, Logstash, Ubuntu]
comments: true
mathjax: true
author: 이희두
---

# PostgreSQL to Elasticsearch Migration 과정

> 본 문서에 사용된 테이블명, 인덱스명, 서버 정보는 실제 운영 환경과 무관한 예시 명칭입니다.

---

## 문서화 목적

- PostgreSQL에 저장된 **로그성 데이터**를 Elasticsearch로 이관할 필요가 있었고,
- 이후 동일한 Migration 작업이 발생할 경우를 대비해 과정을 문서화함

---

## 요약

데이터 마이그레이션 과정에서 **Logstash**를 활용하여 PostgreSQL의 이력성 데이터를  
Elasticsearch에 인덱싱하였다.

대용량 데이터의 효율적인 이동을 위해 **Object Storage**를 중간 스토리지로 활용하였으며,  
CLI 기반 Shell 환경을 통해 CSV 파일 업로드 및 다운로드를 수행하였다.

---

## 고려한 점

- 성능 최적화를 고려하되, **과도한 기술 구성은 지양**
- 운영 DB 부하 최소화
- Migration 과정의 재현성과 안정성 확보

---

## 이관한 데이터 정보

- PostgreSQL
  - 로그성 배터리 데이터가 저장된 **원천 로그 테이블**
  - 기준 시점 약 **40GB 규모**
- Elasticsearch
  - 위 원천 로그 테이블 데이터가 이관된 **로그 분석용 인덱스**

---

## 사용 기술

- Logstash
- Shell 기반 CLI
- Object Storage

---

## 기술 선택 이유

### Logstash

- **CSV 파일을 읽어 Elasticsearch로 인덱싱하는 기능 제공**
- 대용량 데이터 처리에 적합
- Filter를 통해 데이터 전처리 가능
- Elasticsearch와의 높은 호환성으로 추가 개발 최소화

### Object Storage + CLI

- 대용량 CSV 파일을 **안정적이고 빠르게 업로드/다운로드**
- CLI 기반 스크립트 작성으로 자동화 가능
- GUI 환경에서도 스토리지 상태 모니터링 가능

---

## CSV로 Export한 이유  
### 데이터 이관 속도 최적화

- PostgreSQL에서 직접 Logstash 기반 Migration 시 **DB 연결 부하 증가 가능성**
- 대상 테이블은 인덱싱이 되어 있지 않고, 디스크 여유 공간이 부족한 상태
- CSV Export → Batch Migration 방식이 **속도 및 안정성 측면에서 유리**

---

## Migration 순서

### 1️⃣ PostgreSQL → CSV Export

- PostgreSQL 서버 접근
- COPY 명령어를 통해 CSV 파일 생성

```sql
COPY (
  SELECT
    station_id,
    idx,
    device_id,
    snapshot,
    payload,
    battery_id,
    user_id,
    counter,
    metric_1, metric_2, metric_3, metric_4, metric_5,
    metric_6, metric_7, metric_8, metric_9, metric_10,
    voltage,
    current,
    soc,
    balance,
    alarm,
    created_at
  FROM source_log_table
  WHERE created_at >= 'YYYY-MM-DD 00:00:00'
    AND created_at <  'YYYY-MM-DD 00:00:00'
)
TO '/path/to/export.csv'
WITH (FORMAT CSV, HEADER);
````

---

### 2️⃣ CSV 파일 Object Storage 업로드

* scp 또는 CLI를 통해 CSV 파일 이동
* Object Storage에 업로드

```bash
scp -i "ssh-key.pem" user@db-server:/path/export.csv ./export.csv
```

---

### 3️⃣ Object Storage → Elasticsearch 서버 다운로드

* Elasticsearch 서버에서 CLI 환경 구성
* Object Storage로부터 CSV 파일 다운로드

```bash
object-storage-cli get \
  --bucket <bucket-name> \
  --name <csv-file-name> \
  --file ./export.csv
```

---

### 4️⃣ Logstash 파이프라인 구성 및 실행

#### Logstash Pipeline 예시

```conf
input {
  file {
    path => ["/data/export_*.csv"]
    start_position => "beginning"
    sincedb_path => "/dev/null"
    mode => "read"
  }
}

filter {
  csv {
    separator => ","
    columns => [
      "station_id","idx","device_id","snapshot","payload",
      "battery_id","user_id","counter",
      "metric_1","metric_2","metric_3","metric_4","metric_5",
      "metric_6","metric_7","metric_8","metric_9","metric_10",
      "voltage","current","soc","balance","alarm","created_at"
    ]
    skip_header => true
  }

  ruby {
    init => "@@doc_id = ENV['MAX_ID'].to_i + 1"
    code => "
      event.set('doc_id', @@doc_id)
      @@doc_id += 1
    "
  }

  mutate {
    remove_field => ["host","@version","log","event","message"]
  }

  mutate {
    convert => {
      "station_id" => "integer"
      "idx" => "integer"
      "doc_id" => "integer"
      "voltage" => "float"
      "current" => "float"
      "soc" => "float"
    }
  }
}

output {
  elasticsearch {
    hosts => ["http://<elasticsearch-host>:9200"]
    index => "target_log_index"
    document_id => "%{doc_id}"
  }
}
```

---

### Logstash 실행 예시

```bash
MAX_ID=$(curl -s "http://<elasticsearch-host>:9200/target_log_index/_search?size=1&sort=doc_id:desc&_source=doc_id" \
 | jq -r '.hits.hits[0]._source.doc_id // 0')

export MAX_ID

/usr/share/logstash/bin/logstash \
 -f /etc/logstash/logstash.conf \
 -w 1 -b 5000 \
 --pipeline.ecs_compatibility=disabled
```

---

## 고려사항

* CSV 파일을 로컬을 거쳐 Object Storage로 업로드하는 과정은
  DB 서버에 CLI 환경을 직접 구성하면 생략 가능
* PostgreSQL 서버에서 Object Storage로 직접 업로드 시
  Migration 전체 단계 단축 가능

---

## 참고

* Object Storage CLI Setup 문서
* Logstash 공식 문서
  [https://www.elastic.co/guide/en/logstash/current/index.html](https://www.elastic.co/guide/en/logstash/current/index.html)

