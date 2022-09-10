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

### Flattening flows

flow는 비동기적으로 수신된 값 시퀀스를 나타내므로 각 값이 다른 값 시퀀스에 대한 요청을 트리거하는 상황에 쉽게 도달할 수 있습니다.

#### flatMapConcat
```flatMapConcat```은 여러 flow를 연결하는 연산자입니다. flow 간에 연결이 필요한 경우에 사용합니다.

```kotlin
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500) // wait 500 ms
    emit("$i: Second")
}

fun main() = runBlocking<Unit> {
    val startTime = currentTimeMillis() // remember the start time
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms
        .flatMapConcat { requestFlow(it) }
        .collect { value -> // collect and print
            println("$value at ${currentTimeMillis() - startTime} ms from start")
        }
}
```

출력결과를 확인하면 ```flatMapConcat```은 flow를 생성해 또 다른 생성된 flow와 합쳐 하나의 flow로 만들어냅니다.
```
1: First at 118 ms from start
1: Second at 624 ms from start
2: First at 731 ms from start
2: Second at 1233 ms from start
3: First at 1338 ms from start
3: Second at 1840 ms from start
```

#### flatMapMerge 
또다른 병합 모드 연산자로는 ```flatMapMerge```로 모든 흐름을 동시에 수집해서 단일값으로 병합하여 빠르게 방출합니다. 

```kotlin
fun main() = runBlocking<Unit> {
    val startTime = System.currentTimeMillis() // remember the start time
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms
        .flatMapMerge { requestFlow(it) }
        .collect { value -> // collect and print
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")
        }
}
```
아래 출력값을 확인하면 ```flatMapMerge```는 변환을 병렬로 수행함을 알수 있습니다.
```
1: First at 136 ms from start
2: First at 238 ms from start
3: First at 344 ms from start
1: Second at 641 ms from start
2: Second at 744 ms from start
3: Second at 852 ms from start
```

#### flatMapLatest
```flatMapLatest```는 새로운 flow가 방출될 때 직전 flow를 취소합니다. ```collectLatest와 동작이 유사합니다. 

```kotlin
fun main() = runBlocking<Unit> {
    val startTime = System.currentTimeMillis()
    (1..3).asFlow().onEach { delay(100) }
        .flatMapLatest { requestFlow(it) }
        .collect { value ->
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")
        }
}
```

```flatMapLatest```는 새 값이 방출되면 그 실행 블록 ({ requestFlow(it) }) 전체를 취소합니다. 
```
1: First at 133 ms from start
2: First at 239 ms from start
3: First at 345 ms from start
3: Second at 850 ms from start
```



