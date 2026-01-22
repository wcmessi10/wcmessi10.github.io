---
layout: post
title: ResponseEntity
tags: [Spring, HTTP]
comments: true
mathjax: true
author: 이희두
---

## ResponseEntity 개요

`ResponseEntity`는 **HTTP 응답 전체(상태 코드 + 헤더 + 바디)**를 표현하는 Spring 클래스입니다.
컨트롤러에서 반환함으로써 **HTTP 상태 코드와 응답 데이터를 명확하게 제어**할 수 있습니다.

---

## 자주 사용하는 ResponseEntity 메서드

* **`ResponseEntity.status(HttpStatus)`**
  → 주어진 HTTP 상태 코드를 갖는 응답을 생성

  ```java
  return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("error");
  ```

* **`ResponseEntity.ok()`**
  → HTTP 상태 코드 **200 (OK)**

  ```java
  return ResponseEntity.ok(result);
  ```

* **`ResponseEntity.notFound()`**
  → HTTP 상태 코드 **404 (Not Found)**

  ```java
  return ResponseEntity.notFound().build();
  ```

* **`ResponseEntity.badRequest()`**
  → HTTP 상태 코드 **400 (Bad Request)**

  ```java
  return ResponseEntity.badRequest().body("invalid request");
  ```

* **`ResponseEntity.created(URI)`**
  → HTTP 상태 코드 **201 (Created)**
  → `Location` 헤더에 생성된 리소스의 URI를 포함

  ```java
  URI location = URI.create("/users/1");
  return ResponseEntity.created(location).build();
  ```

이 외에도 `noContent()`, `accepted()`, `forbidden()` 등 REST API에서 자주 쓰이는 상태 코드들을 편리하게 제공합니다.

---

## `ResponseEntity<?>`에서 `?`의 의미

`ResponseEntity<?>`의 `?`는 **와일드카드 제네릭 타입**을 의미합니다.

즉,

> “응답 본문(body)의 타입이 아직 정해지지 않았거나, 여러 타입이 올 수 있다”

라는 뜻입니다.

### 왜 `?`를 쓰는가?

`ResponseEntity<T>`에서 `T`는 **응답 본문의 타입**입니다.

```java
ResponseEntity<String>
ResponseEntity<UserDto>
ResponseEntity<List<UserDto>>
```

처럼 구체적인 타입을 지정할 수도 있지만,

```java
ResponseEntity<?>
```

로 선언하면,

* 성공 시: 데이터 객체
* 실패 시: 에러 메시지
* 혹은 바디가 없는 응답

등 **여러 형태의 응답을 유연하게 처리**할 수 있습니다.

---

## 실무에서의 사용 예시

```java
public ResponseEntity<?> getUser(Long id) {
    Optional<User> user = userService.findById(id);

    if (user.isEmpty()) {
        return ResponseEntity.notFound().build();
    }

    return ResponseEntity.ok(user.get());
}
```

위 코드에서:

* 성공 시 → `User` 객체 반환
* 실패 시 → 바디 없는 404 응답

이처럼 **하나의 메서드에서 서로 다른 응답 타입을 반환해야 할 때**
`ResponseEntity<?>`가 가장 자연스러운 선택입니다.

---

## 정리 한 줄 요약

* `ResponseEntity`는 **HTTP 상태 코드 + 헤더 + 바디를 명확히 제어**하기 위한 클래스
* `ResponseEntity<?>`의 `?`는 **응답 바디 타입을 유연하게 처리하기 위한 와일드카드**
* REST API에서 **성공/실패 케이스가 섞일 때** 특히 유용

