---
title: "Kotlin Flow 연산자"
excerpt: "Kotlin Flow 여러가지 연산자"

categories:
  - Kotlin
tags:
  - [Kotlin]

permalink: /kotlin/kotlin-flow/

toc: true
toc_sticky: true

date: 2022-09-05
last_modified_at: 2022-09-05
---

# Asynchronous Flow

## Representing multiple values
컬렉션을 사용하여 Kotlin에서 여러 값을 나타낼 수 있습니다. 
예를 들어, 세 개의 숫자 목록을 반환한 다음 forEach를 사용하여 모두 인쇄하는 간단한 함수를 가질 수 있습니다.

```kotlin

fun collectionsRepresent(): List<Int> = listOf(1, 2, 3)

fun main() {
    collectionsRepresent().forEach { value ->
        println(value)
    }
}

```

```
1
2
3
```

일반적인 상황에서 for문의 성능이 forEach보다 좋으나 Collection을 상속받고 있는 자료구조는 
해당 클래스들에 대한 몇몇 함수들은 inline을 제공하게 됩니다. 그렇기에 for문 보다 더 좋은 성능을 제공합니다.


