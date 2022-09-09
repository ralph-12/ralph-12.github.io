---
title: "Kotlin Flows 알아보기 2"
excerpt: "가깝고 먼 Kotlin Flows 알아보기 2편"

categories:
  - Kotlin
tags:
  - [Kotlin]

permalink: /kotlin/kotlin-flow-2/

toc: true
toc_sticky: true

date: 2022-09-09
last_modified_at: 2022-09-09
---

### Flow context
flow 수집은 항상 호출 coroutine의 context에서 발생합니다. 
예를 들어, ```simple``` flow가 있는 경우 다음 코드는 ```simple``` flow의 구현 세부 정보에 관계없이 이 코드 작성자가 지정한 context에서 실행됩니다.

flow의 이 속성을 context 보존이라고 합니다.

따라서 기본적으로 flow { ... } 빌더의 코드는 해당 flow의 수집기가 제공하는 context에서 실행됩니다.

아래는 호출된 스레드를 인쇄하고 세 개의 숫자를 내보내는 ```simple``` 함수입니다.

```kotlin
fun simple(log: String): Flow<Int> = flow {
    println("[${Thread.currentThread().name}] Started $log flow")
    for (i in 1..3) {
        emit(i)
    }
}

fun main() = runBlocking {
    simple("simple").collect { value ->
        println("[${Thread.currentThread().name}] Collected $value")
    }
}
```

```
Started simple flow
[main] Started simple flow
[main] Collected 1
[main] Collected 2
[main] Collected 3
```

simple().collect는 메인 쓰레드에서 호출되기 때문에 simple의 flow 바디도 메인 쓰레드에서 호출됩니다.
이것은 실행 context를 신경 쓰지 않고 호출자를 차단하지 않는 빠르게 실행되는 또는 비동기 코드에 대한 완벽한 기본값입니다.
