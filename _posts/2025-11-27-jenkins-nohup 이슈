---
layout: post
title: Jenkins 파이프라인 nohup 오류 원인 및 해결 방법
tags: [Backend, Spring, Cache]
comments: true
mathjax: true
author: 이희두
---

## 문제 개요

Jenkins 파이프라인 실행 시, 가장 초기 단계(`Webhook Start Notify` 등)에서 다음 오류와 함께 빌드가 즉시 실패함:

```
java.io.IOException: error=0, Failed to exec spawn helper: pid: xxxx, exit value: 1
Caused: java.io.IOException: Cannot run program "nohup"
(in directory "..."): error=0, Failed to exec spawn helper

```

또한 전체 Stage가 다음과 같이 처리되지 않고 모두 Skip 처리됨:

```
Stage "XXXXX" skipped due to earlier failure(s)

```

---

## 문제 원인

Jenkins의 `sh` 스텝은 내부적으로 **durable-task 플러그인**을 사용하며, Linux에서 비동기 실행을 위해 `nohup`을 자동 사용한다.

일부 Linux + Java 조합에서 Java의 `posix_spawn()` 메커니즘이 `spawn helper` 오류를 발생시키며, 이로 인해 Jenkins 내부적으로 `nohup` 실행이 실패한다.

> 즉, Jenkinsfile 문제도 아님, nohup 파일 문제도 아님, 자원 부족도 아님.
> 
> 
> **Java JVM 프로세스 생성 방식과 Linux 환경 조합에서 발생하는 버그**이다.
> 

---

## 해결 방법

Java에서 프로세스 실행 방식을 **vfork 방식으로 강제**하면 문제 해결됨.

### 1) Jenkins 설정 파일 수정

편집기 열기 (nano 추천):

```bash
sudo nano /etc/default/jenkins

```

다음 옵션이 포함된 줄을 찾는다:

```
JAVA_ARGS="-Djava.awt.headless=true"

```

아래처럼 **옵션을 끝에 추가**한다:

```
JAVA_ARGS="-Djava.awt.headless=true -Djdk.lang.Process.launchMechanism=vfork"

```

### 2) Jenkins 재시작

```bash
sudo systemctl restart jenkins

```

---

## 문제 재현 테스트 & 해결 확인

### 테스트용 Jenkins 파이프라인 실행

```groovy
pipeline {
    agent any
    stages {
        stage('test') {
            steps {
                sh 'echo hello'
            }
        }
    }
}

```

**정상 실행될 경우 → 기존 파이프라인도 정상 작동함.**

---

## 추가 참고

| 항목 | TRUE / FALSE |
| --- | --- |
| nohup 파일 없음 문제인가? | ❌ 아님 (/usr/bin/nohup 정상 존재) |
| Jenkins 유저 권한 문제인가? | ❌ 아님 |
| OS 메모리 부족인가? | ❌ 아님 |
| durable-task + Java spawn 버그인가? | ✔ YES |

---

## 추후 권장 조치

- Jenkins 및 durable-task plugin 최신 버전 유지
- 불필요한 `nohup`, 백그라운드 실행 로직은 Jenkinsfile에서 제거 권장
    
    → 서비스 실행은 systemd/docker/k8s 등이 담당하는 것이 안정적
    

---

## 결론

> Jenkins의 Java 프로세스 생성 방식(posix_spawn)에 의한 nohup 실행 실패 버그로 인해 파이프라인 실패가 발생했으며, JVM 옵션 -Djdk.lang.Process.launchMechanism=vfork 추가를 통해 해결했다.
>
