---
layout: post
title: Kotlin의 Statement와 Expression
tags: [Kotlin, Statement, Expression]
comments: true
mathjax: true
author: 이희두
---


Kotlin을 공부하면서, Statement와 Expression에 대해 좀 더 심층적으로 알고 있다면 더욱더 학습 능력이 향상될 것 같아 문서로 남긴다.

## Statement와 Expression

Statement는 문장을 실행하는 단위인 행위 중심이고 Expression은 값을 반환하는 단위 값 중심이다. 이 둘은 Statement 안에 Expression이 들어가는 포함관계다. 프로그램의 흐름을 제어하거나 상태를 변경하면 Statement이고, 거기서 값을 반환하면 Statement이자 Expression이다. 

## Java와 Kotlin의 Statement, Expression 해석 차이

앞에서 말했다시피 값을 만들어내는 구문이면 Expression이라고 했다.

Java에서 if, switch, try들은 실행 흐름 제어용 구문이라 Expression이 아닌 Statement로 취급하고 있으나, Kotlin에서 if, when, try에서 값을 반환할 수 있다면 Expression이다.

- 대신 Java에는 삼항연산자가 있으나 Kotlin은 없다고 한다.

```java
// Java의 if문을 활용한 Statement 구문
int max;
if (a > b) {
	max = a;
} else { 
	max = b;
}
```

```kotlin
// Kotlin의 if문을 활용한 Expresssion 구문
val max = if (a > b) a else b;

// Kotlin의 try catch문을 활용한 Expression 구문
fun plusFive(str: String): Int = 
	try {
		str.toInt() + 5
	} catch (e: NumberFormatException) {
	  5      // catch문에도 같은 타입 값을 반환해야한다.
  } finally {
	  println("Plus Five") // finally는 값이랑 상관없다.
  }
```

## 주의사항

- 반복문은 Kotlin에서는 Statement이다.
- 제어문으로 값을 반환할 때는 무조건 타입이 같아야한다.
- 마지막 줄에 값을 반환하면 앞에 여러 실행 절차를 거쳐도 된다. - Block Expression
- if문으로 반환하면 else문으로 반환도 해줘야한다
- 블록이 너무 길어지면 가독성이 떨어진다.

## Kotlin이 Expression에 더 중요시하는 이유

Kotlin 자체의 철학이 Expression Oriented Programming을 지향한다.

- Side effect 최소화 및 예측 가능성 증
- 함수형 스타일 강화
- 모드 간결성 강화

## 정리

- Kotlin은 Java와 다르게 Expression 중심의 언어이다.
