---
layout: post
title: Presigned URL
tags: [Bucket, Cloud, Presigned URL, File Approach]
comments: true
mathjax: true
author: 이희두
---

## Presigned URL이란?

클라우드 권한이 있는 서버가 보안 자격 증명 없이 특정 버킷에 제한된 시간 동안 접근할 수 있도록 생성해주는 임시 URL이다.

서버를 통해 버킷에 접근하는 방식이 아닌 안전하게 직접 버킷에 접근하는 방식이라 서버 부하를 줄이고 보안을 강화할 수 있다.

## Presigned URL을 알게 되고 도입하게 된 배경

기존에는 파일 업로드 시 API Gateway를 거쳐 서버로 전달하는 구조를 사용했다.

그런데 최근 한 번의 요청에서 Gateway 허용 용량을 초과하는 파일 업로드가 발생하면서 문제가 생겼다.

이를 계기로 대용량 파일 업로드 처리 방법을 찾다가 Presigned URL 방식을 도입하게 되었다.

## 사용한 라이브러리

필자는 OCI 클라우드를 사용중이다. 다른 모듈들도 사용중이라 shaded-full 버전을 사용중이다.

```
implementation 'com.oracle.oci.sdk:oci-java-sdk-shaded-full'

// 아래 라이브러리만 사용해도 Presigned URL 기능을 활용할 수 있다.
implementation "com.oracle.oci.sdk:oci-java-sdk-objectstorage"
implementation "com.oracle.oci.sdk:oci-java-sdk-common"
```

## 적용 코드

```java
public PresignedUrlResponse generatePresignedUrl(String originalFilename) {
        String extension = originalFilename.substring(originalFilename.lastIndexOf('.') + 1).toLowerCase();
        String filenameWithoutExtension = originalFilename.substring(0, originalFilename.lastIndexOf('.'));
        String objectName = filenameWithoutExtension + "_" + Instant.now().toEpochMilli() + "." + extension;

        CreatePreauthenticatedRequestDetails details = CreatePreauthenticatedRequestDetails.builder()
                .name("PresignedUrl")
                .accessType(CreatePreauthenticatedRequestDetails.AccessType.ObjectReadWrite)
                .objectName(objectName)
                .timeExpires(Date.from(Instant.now().plusSeconds(3600))) // 1시간 유효
                .build();

        CreatePreauthenticatedRequestRequest request = CreatePreauthenticatedRequestRequest.builder()
                .namespaceName(namespaceName)
                .bucketName(bucketName)
                .createPreauthenticatedRequestDetails(details)
                .build();

        CreatePreauthenticatedRequestResponse response = client.createPreauthenticatedRequest(request);

        // OCI Object Storage의 PAR(Pre-Authenticated Request) URL 형식:
        // https://objectstorage.<region>.oraclecloud.com/p/<id>/n/<namespace>/b/<bucket>/o/<object>
        String domain = "objectstorage.ap-singapore-1.oraclecloud.com";
        if (url != null && url.contains("objectstorage")) {
            try {
                java.net.URL ociUrl = new java.net.URL(url);
                domain = ociUrl.getHost();
            } catch (Exception e) {
                log.warn("Failed to parse oci.url, using default domain", e);
            }
        }

        String uploadUrl = "https://" + domain + response.getPreauthenticatedRequest().getAccessUri();
        String fileUrl = url + objectName;

        return PresignedUrlResponse.builder()
                .uploadUrl(uploadUrl)
                .fileUrl(fileUrl)
                .build();
  }
```

## Presigned URL을 이용한 파일 업로드 흐름

1. 클라이언트 → 서버 - 파일명과 Presigned URL 요청
2. 서버 → 클라이언트 - 반환값으로 Presigned URL, File URL 제공
3. 클라이언트 → 버킷 - Presigned URL 통해 파일 업로드 (Put 요청 바이너리로 파일을 업로드해야한다)
4. 클라이언트 → 서버 - File URL을 DB에 저장하는 로직 요청

## Presigned URL 주의사항

- Presigned URL은 유출되면 누구든 해당 시간동안 접근이 가능하다
- 만료 시간이 너무 짧으면 업로드 중 만료된다
- 클라이언트와 스토리지 거리가 멀거나 네트워크 품질 문제가 있을 시 업로드 속도가 느려질 수 있다
- 고아 객체가 발생할 수 있다 - 삭제 전략 필요

## 결론

Presigned URL 방식은 대용량 파일 업로드 환경에서 **API Gateway와 서버의 부담을 줄이면서도 보안을 유지할 수 있는 실용적인 아키텍처 패턴**이다.

특히 다음과 같은 상황에서 효과적이다.

- 대용량 파일 업로드가 빈번한 서비스
- API Gateway 용량/타임아웃 제약이 있는 환경
- 서버 리소스를 업로드 트래픽에 소모하고 싶지 않은 경우
- 클라이언트-스토리지 직통 업로드 구조가 가능한 서비스

다만, 만료 시간 관리, 고아 객체 정리, URL 유출 방지 등 **운영 레벨의 설계가 함께 고려되어야 한다.**

결과적으로 Presigned URL은 단순한 업로드 기법이 아니라,

**트래픽 구조를 바꾸는 아키텍처 선택지**에 가깝다.

대용량 업로드가 있는 서비스라면 한 번쯤 도입을 검토할 가치가 충분하다.
