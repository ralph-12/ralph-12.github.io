---
title: "Coroutine의 취소와 타임아웃"
excerpt: "Coroutine의 취소와 타임아웃"

categories:
  - Kotlin
tags:
  - [Kotlin]

permalink: /kotlin/kotlin-coroutine-cancel-timeout/

toc: true
toc_sticky: true

date: 2022-10-08
last_modified_at: 2022-10-09
---

## 명시적인 ```Job```에 대해 취소 하기

```Job```은 생성, 실행 중, 실행 완료, 취소 중, 취소 완료라는 총 5가지의 상태를 가지고 있습니다.

```Job```을 취소하는 방법은 매우 간단합니다. 바로 ```cancel()```를 사용 하면 되는데 아래의 예제를 확인해 보겠습니다. 

``` kotlin
suspend fun jobCancelExample() = coroutineScope {
    val job1 = launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("1!")
    }

    val job2 = launch {
        println("launch2: ${Thread.currentThread().name}")
        println("2!")
    }

    val job3 = launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("3!")  
    }

    delay(800L)
    job1.cancel()
    job2.cancel()
    job3.cancel()
    println("4!")
}

fun main() = runBlocking {
    jobCancelExample()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}
``` 
