---
layout: post
title: Logstash 통해 Slack 메시지 UDP 전송 테스트
tags: [Slack bot, Logstash]
comments: true
mathjax: true
author: 이희두
---

# 목적

클라이언트에서 발생하는 앱 특정 로그들을 Slack에 기록하고자 테스트 진행

---

# Slack App 설정

Slack 봇을 이용해 채널 메시지를 출력하기 위해 Workspace에 App 생성

![Screenshot](/assets/img/스크린샷_2024-08-20_162121 (1).png)

---

# Slack `chat.postMessage` API

- 특정 채널로 메시지를 보내는 Slack Web API
- `chat.postMessage` 사용을 위해 `chat:write` 권한(스코프) 추가 필요

(스크린샷 생략)

## 의존성 추가

```gradle
implementation("com.slack.api:slack-api-client:<version>")
````

Slack API 호출을 위해 SDK를 활용

## Slack 호출 코드 예시

```java
@Autowired
private SlackComponent slackComponent;

public ChatPostMessageResponse message(String channelId, String message)
        throws SlackApiException, IOException {

    Slack slack = Slack.getInstance();
    return slack.methods().chatPostMessage(req -> req
            .token(slackComponent.token)
            .channel(channelId)
            .text(message));
}
```

* 요청 값:

  * `token` (Bot User OAuth Token)
  * `channelId` (채널 ID)
  * `message` (보낼 메시지)
* Bot User OAuth Token은 서버 리소스/환경변수 등으로 안전하게 관리

---

# 서버 컨트롤러

```java
@Slf4j
@RestController
@RequestMapping("/<internal-api-prefix>/slack")
public class SlackController {

    @Autowired
    SlackService slackService;

    @GetMapping("/message")
    public ResponseEntity<?> message(@RequestParam String message)
            throws SlackApiException, IOException {

        return ResponseEntity.ok(slackService.message("<channel-id>", message));
    }
}
```

* 채널 ID는 테스트 단계에서 고정값 사용
* 컨트롤러 단위 사전 테스트 완료

(스크린샷 생략)

---

# Logstash 설정

클라이언트에서 UDP 형태로 Logstash로 메시지를 전송하고,
Logstash가 이를 받아 서버 컨트롤러를 호출하도록 구성

* 새로운 `.conf` 파일 생성
* UDP 입력 포트는 예시로 `5034` 사용
* input JSON 형태: `{"message":"<text>"}`

구성 요약:

* **input**: UDP 수신
* **filter**: Ruby 코드로 UTF-8 인코딩/정리
* **output**: HTTP output 플러그인으로 서버 컨트롤러 호출

(스크린샷 생략)

추가로 pipeline에 해당 설정 파일을 등록

---

# Logstash 테스트

* PowerShell 등으로 UDP 메시지 전송
* Logstash 수신 로그 확인
* Slack 채널 메시지 업로드 확인

(스크린샷 생략)

---

# 참고

* [https://api.slack.com/methods/chat.postMessage](https://api.slack.com/methods/chat.postMessage)
* [https://api.slack.com/scopes/chat:write](https://api.slack.com/scopes/chat:write)
* [https://www.elastic.co/guide/en/logstash/current/plugins-outputs-http.html](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-http.html)
* [https://www.elastic.co/guide/en/logstash/current/plugins-filters-ruby.html](https://www.elastic.co/guide/en/logstash/current/plugins-filters-ruby.html)
