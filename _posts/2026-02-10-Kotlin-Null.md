---
layout: post
title: Kotlin의 null 처리
tags: [NPE, Kotlin, Null-safe]
comments: true
mathjax: true
author: 이희두
---

Kotlin을 학습하면서 Kotlin의 null 처리에 대해 상당히 엄격한 점이 Java base 개발자인 나에겐 재미있어 글로 남기고자 한다

## Kotlin이 null에 엄격한 이유

- Kotlin은 Null을 타입 시스템에 포함하는 철학을 가지고 있다. 타입 레벨에서 Null을 관리한다.
    - 아예 타입을 Non-null 타입, Nullable 타입 2가지로 나뉜다.

## Non-null 타입

null을 허용하지 않는 타입이 된다.

```kotlin
val test: Double = 0.0   // 가능
val test2: Double = null // 컴파일 에러
```

## Nullable 타입

?가 붙으면 null을 허용하는 다른 타입이 된다.

```kotlin
val test: Double? = 0.0   // 가능
val test2: Double? = null // 가능
```

## 컴파일러에서부터 막히는 NPE

Kotlin은 NPE 자체를 컴파일 단계에서 대부분 막힌다.

```kotlin
val test :String = null   // 해당 코드는 Non-null 타입으로 실행 자체가 안된다.
val test2 :String? = null // 해당 코드는 Nullable 타입 코드로 실행 자체가 가능하다.

println(test2.length)     // 해당 코드는 NPE를 발생하기 때문에 실행 자체를 할 수 없다.

```

## 안전한 Null 처리

Kotlin에서는 안전한 Null 처리를 위해 다음과 같은 기능을 제공한다.

## Safe call

객체의 프로퍼티나 메소드를 사용할 때, Null일 경우 그냥 Null을 반환하여 NPE 발생을 방지하기 위한 Null-safe 처리

함수나 프로퍼티 앞에 ?를 붙여준다.

```kotlin
val test: String?
test = "이희두"
return if (test.substring(0, 3) == "이희두"){ // Nullable에 대해 대응을 안해놓은 상태로 Null-Safe가 아니므로 불가능
	"이희두"
} else {
	"딴사람"
}

return if (test?.substring(0, 3) == "이희두"){ // Safe call을 활용하여 Nullable에 대응을 해놓았기 때문에 Null-safe로 가능
	"이희두"
} else {
	"딴사람"
}

```

## Elvis 연산자

앞의 결과가 null일 경우 뒤의 값을 사용하는 연산자

```kotlin
val test: Int? = null
val test2 = (test ?: 0) + 5 // test가 null이면 0으로 바꿔달라는 Elvis 연산자로 Null-Safe 처리
```

## Null-safe의 장점

1. 런타임 중 NPE 대폭 감소
2. 코드의 의도 명확

## 결론

Kotlin은 Null을 별도로 타입으로 관리한다.
