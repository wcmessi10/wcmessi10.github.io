---
layout: post
title: Batch 메타데이터 테이블에서 Spring Batch의 Job, Step 정상동작 확인 DML
tags: [Spring Batch, Meata Data]
comments: true
mathjax: true
author: 이희두
---

## 작성 사유

Spring Batch의 Job 추가로 인해 스케쥴러 정상동작 확인이 필요할 경우 참고 목적

## Spring Batch 메타데이터 테이블

Spring Batch는 기본적으로 아래 6개의 테이블을 사용합니다

| 테이블명 | 설명 | 주요 컬럼 |
| --- | --- | --- |
| **BATCH_JOB_INSTANCE** | Job 이름과 파라미터 조합으로 생성된 인스턴스 정보 | `JOB_INSTANCE_ID`, `JOB_NAME` |
| **BATCH_JOB_EXECUTION** | Job 실행 정보 (실제 실행 상태) | `JOB_EXECUTION_ID`, `STATUS`, `START_TIME`, `END_TIME`, `EXIT_CODE` |
| **BATCH_JOB_EXECUTION_PARAMS** | Job 실행 시 전달된 파라미터 | `KEY_NAME`, `STRING_VAL`, `DATE_VAL` 등 |
| **BATCH_STEP_EXECUTION** | Step 단위 실행 정보 | `STEP_EXECUTION_ID`, `STEP_NAME`, `STATUS`, `READ_COUNT`, `WRITE_COUNT` |
| **BATCH_STEP_EXECUTION_CONTEXT** | Step 실행 중 컨텍스트(상태값 등) | `SHORT_CONTEXT`, `SERIALIZED_CONTEXT` |
| **BATCH_JOB_EXECUTION_CONTEXT** |  |  |

## 테이블 조회

### 1. **최근 실행 Job 확인**

```sql
SELECT JOB_EXECUTION_ID, JOB_INSTANCE_ID, STATUS, START_TIME, END_TIME, EXIT_CODE
FROM BATCH_JOB_EXECUTION
ORDER BY JOB_EXECUTION_ID DESC;

```

- `STATUS` = `COMPLETED` → 정상 완료
- `FAILED` → 실패
- `STARTING`, `STARTED` → 실행 중

### 2. **어떤 Job이 실행됐는지 확인**

```sql
SELECT ji.JOB_NAME, je.STATUS, je.START_TIME, je.END_TIME, je.EXIT_CODE
FROM BATCH_JOB_INSTANCE ji
JOIN BATCH_JOB_EXECUTION je ON ji.JOB_INSTANCE_ID = je.JOB_INSTANCE_ID
ORDER BY je.START_TIME DESC;
```

### 3. **실행된 Step 상태 확인**

```sql
SELECT STEP_NAME, STATUS, READ_COUNT, WRITE_COUNT, COMMIT_COUNT, EXIT_CODE
FROM BATCH_STEP_EXECUTION
WHERE JOB_EXECUTION_ID = {확인한 JOB_EXECUTION_ID};
```

### 4. **Job Parameter 확인**

```sql
SELECT * FROM BATCH_JOB_EXECUTION_PARAMS
WHERE JOB_EXECUTION_ID = {확인한 JOB_EXECUTION_ID};

```

## 추가 참고 사항

- 해당 테이블로는 리스너 확인은 어렵다
-
