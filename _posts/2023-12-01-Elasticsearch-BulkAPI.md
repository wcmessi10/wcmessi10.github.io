---
layout: post
title: Elasticsearch Bulk API
tags: [Elasticsearch, Bulk]
comments: true
mathjax: true
author: 이희두
---

## Elasticsearch Bulk API 개념 정리

**Bulk API**는 *단일 HTTP 요청*으로 여러 개의 문서를 **색인(index)** 하거나 **업데이트(update)**, **삭제(delete)** 할 수 있는 기능입니다.
요청을 한 번만 보내기 때문에, 여러 번 요청할 때 발생하는 **네트워크 오버헤드가 줄어** 대량 처리 성능이 크게 좋아집니다.

### NDJSON(Newline Delimited JSON) 포맷이 핵심

Bulk API 요청 본문은 일반적인 JSON 배열 형태가 아니라, **줄바꿈(\n)으로 구분되는 NDJSON** 형식입니다.

* 각 작업은 `action/meta` 라인 + (필요한 경우) `source/doc` 라인으로 구성됩니다.
* 모든 라인은 `\n`으로 끝나야 하며, **마지막 줄도 개행이 들어가는 게 안전**합니다(실무에서 자주 실수나는 포인트).

예시:

```json
{ "index" : { "_index" : "myindex", "_id" : "1" } }
{ "field1" : "value1" }

{ "delete" : { "_index" : "myindex", "_id" : "2" } }

{ "update" : { "_index" : "myindex", "_id" : "3" } }
{ "doc" : { "field2" : "updated_value" } }
```

### 작업 유형

* `index` : 문서를 색인(없으면 생성, 있으면 덮어쓰기)
* `delete` : 문서 삭제
* `update` : 문서 일부 업데이트(보통 `doc` 사용)

추가로 자주 쓰는 작업:

* `create` : **동일 ID가 이미 존재하면 실패**(“있으면 안 된다”가 비즈니스 룰일 때 유용)

---

## “JPA 스타일 Bulk”로 구현한다는 의미

JPA의 `saveAll()`처럼, 애플리케이션 입장에서는 “여러 문서를 모아서 한 번에 처리”하고 싶습니다.
Elasticsearch에서 그 역할을 해주는 게 Bulk API(또는 Bulk Processor)입니다.

즉, 구현 방향은 보통 이렇게 됩니다.

1. (DB 조회 등으로) 저장할 문서 리스트를 만든다
2. 문서 리스트를 BulkRequest(또는 NDJSON 문자열)로 구성한다
3. Bulk API로 한 번에 전송한다
4. 응답(BulkResponse)을 순회하며 성공/실패를 기록하거나 재시도 정책을 적용한다

---

## Bulk Processor 개념

**Bulk Processor**는 요청을 즉시 보내지 않고, 내부 버퍼에 모아두었다가 조건에 따라 자동으로 flush(전송)하는 방식입니다.

대표적인 flush 조건:

* 일정 개수 이상 누적
* 일정 바이트 이상 누적
* 일정 시간이 지나면 주기적으로 flush
* 동시 실행(concurrent requests) 설정

대량 인덱싱이나 로그성 데이터 적재에서 특히 많이 씁니다.

---

## Java 클라이언트 기반 BulkRequest 예시

```java
BulkRequest bulkRequest = new BulkRequest();
bulkRequest.add(new IndexRequest("myindex").id("1").source("field1", "value1"));
bulkRequest.add(new DeleteRequest("myindex").id("2"));
bulkRequest.add(new UpdateRequest("myindex", "3").doc("field2", "updated_value"));
BulkResponse bulkResponse = client.bulk(bulkRequest, RequestOptions.DEFAULT);
```

실무에서는 `bulkResponse.hasFailures()` 체크 후, 실패한 item을 찾아 **로그 + 재시도(선택)** 로 이어지는 패턴이 흔합니다.

---

## DocWriteResponse란?

`DocWriteResponse`는 Elasticsearch에서 **문서 단위 쓰기 작업 결과**를 표현하는 응답 클래스입니다.

* 하위 타입:

  * `IndexResponse`
  * `UpdateResponse`
  * `DeleteResponse`

이 응답에서 보통 확인하는 정보:

* 성공/실패 여부(또는 상태)
* 문서 ID
* version
* 생성(created)인지 업데이트(updated)인지 여부 등

Bulk에서는 각 item마다 이런 결과가 들어오기 때문에, “BulkResponse → item별 response” 형태로 확인합니다.

---

## 버전 주의: Spring Data Elasticsearch 7.x / 8.x 이슈 메모 (표현 개선 포인트)

여기서 한 가지는 문장 표현을 약간 조정하는 걸 추천해요.

* Spring Data Elasticsearch를 사용할 경우 **프로젝트에서 사용하는 Spring Data Elasticsearch 버전과 Elasticsearch 서버 버전의 호환성을 반드시 확인해야 한다.**
* 조합이 안 맞으면 “조회는 되는데 인덱싱/벌크가 깨짐” 같은 현상이 나올 수 있습니다.(사용자 경험상 많이 발생).
