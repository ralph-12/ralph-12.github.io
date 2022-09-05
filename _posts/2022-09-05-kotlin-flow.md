---
title: "Kotlin Flows 알아보기"
excerpt: "가깝고 먼 Kotlin Flows 알아보기"

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

## Asynchronous Flow
suspand 함수는 단일 값을 비동기적으로 반환하지만 어떻게 여러개의 값을 반환할 수 있을까요?
Kotlin Flows가 이를 해결해 줍니다.


### Representing multiple values
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


### Suspending functions
이러한 값이 비동기 코드로 계산될 때 suspend 수정자로 간단한 함수를 표시할 수 있으므로
차단 없이 작업을 수행하고 결과를 목록으로 반환할 수 있습니다.

```kotlin
suspend fun simple(): List<Int> {
    delay(1000)
    return listOf(1, 2, 3)
}

fun main() = runBlocking {
    simple().forEach() { value ->
        println(value)
    }
}
```

```
1
2
3
```

이 코드는 1초 동안 기다린 후 숫자를 인쇄합니다.


### Flows
List<Int> 결과 유형을 사용하면 한 번에 모든 값만 반환할 수 있다는 의미입니다.   
비동기식으로 계산되는 값의 스트림을 나타내기 위해 동기식으로 계산된 값에 Sequence<Int> 유형을 사용하는 것처럼 
Flow<Int> 유형을 사용할 수 있습니다.

```kotlin
fun flowsSimple(): Flow<Int> = flow { // flow builder
    for (i in 1..3) {
        delay(100) // pretend we are doing something useful here
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    // Launch a concurrent coroutine to check if the main thread is blocked
    launch {
        for (k in 1..3) {
            println("I'm not blocked $k")
            delay(100)
        }
    }

    // Collect the flow
    flowsSimple().collect { value ->
        println(value)
    }
}
```
이 코드는 메인 스레드를 차단하지 않고 각 숫자를 인쇄하기 전에 기다린 후 메인 스레드에서 실행되는 별도의 코루틴에서
100ms마다 "I'm not locked"를 인쇄하여 확인합니다.

```
I'm not blocked 1
1
I'm not blocked 2
2
I'm not blocked 3
3
```
