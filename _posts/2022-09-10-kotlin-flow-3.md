---
title: "Kotlin Flows 알아보기 3"
excerpt: "가깝고 먼 Kotlin Flows 알아보기 3편"

categories:
  - Kotlin
tags:
  - [Kotlin]

permalink: /kotlin/kotlin-flow-3/

toc: true
toc_sticky: true

date: 2022-09-10
last_modified_at: 2022-09-10
---


### Composing multiple flows 

여러 흐름을 구성하는 다양한 방식이 있습니다. 

#### Zip

Kotlin 표준 라이브러리의 ```Sequence.zip``` 확장 함수와 마찬가지로 flow에는 두 flow의 해당 값을 결합하는 ```zip``` 연산자가 있습니다.

```kotlin
fun main() = runBlocking {
    val nums = (1..3).asFlow() // numbers 1..3
    val strs = flowOf("one", "two", "three") // strings

    nums.zip(strs) { a, b -> "$a -> $b" } // compose a single string
        .collect { println(it) }
}
```

```
1 -> one
2 -> two
3 -> three
```

#### Combine

flow이 변수 또는 연산의 가장 최근 값을 나타낼 때(Conflation 참조), 해당 flow의 가장 최근 값에 따라 계산을 수행하고 업스트림 중 하나가 발생할 때마다 이를 다시 계산해야 할 수 있습니다. 해당 연산자를 combine이라고 합니다. 

아래의 예제는 위의 ```zip``` 연산자를 다시 구현해보았습니다. 단 ```zip``` 연산자를 사용한 결과가 400ms마다 출력됩니다.

```kotlin
fun main() = runBlocking<Unit> {
    val nums = (1..3).asFlow().onEach { delay(300) }
    val strs = flowOf("one", "two", "three").onEach { delay(400) }
    val startTime = System.currentTimeMillis()
    nums.zip(strs) { a, b -> "$a -> $b" }
        .collect { value -> //collect and print
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")
        }
}
```

```
1 -> one at 416 ms from start
2 -> two at 821 ms from start
3 -> three at 1234 ms from start
```

```zip``` 연산자 대신 ```combine```를 사용하면 어떻게 될까요?

```kotlin
fun main() = runBlocking<Unit> {
    val nums = (1..3).asFlow().onEach { delay(300) }
    val strs = flowOf("one", "two", "three").onEach { delay(400) }
    val startTime = System.currentTimeMillis()
    nums.combine(strs) { a, b -> "$a -> $b" } // compose a single string with "combine"
        .collect { value -> //collect and print
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")
        }
}
```

```
1 -> one at 424 ms from start
2 -> one at 630 ms from start
2 -> two at 830 ms from start
3 -> two at 936 ms from start
3 -> three at 1236 ms from start
```

