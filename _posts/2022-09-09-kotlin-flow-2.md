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


### Wrong emission withContext

그러나 ```long-running CPU-consuming``` 코드는 Dispatchers.Default의 context에서 실행되어야 하고 UI 업데이트 코드는 Dispatchers.Main의 context에서 실행되어야 할 수 있습니다. 일반적으로 withContext는 Kotlin coroutine을 사용하여 코드의 context를 변경하는 데 사용되지만 flow { ... } 빌더의 코드는 context 보존 속성을 준수해야 하며 다른 context에서 내보내는 것이 허용되지 않습니다.

```kotlin
fun wrongFlowSimple(): Flow<Int> = flow {
    // The WRONG way to change context for CPU-consuming code in flow builder
    kotlinx.coroutines.withContext(Dispatchers.Default) {
        for (i in 1..3) {
            Thread.sleep(100) // pretend we are computing it in CPU-consuming way
            emit(i) // emit next value
        }
    }
}

fun main() = runBlocking<Unit> {
    wrongFlowSimple().collect { value -> println(value) }
}
```

```
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
		Flow was collected in [BlockingCoroutine{Active}@72e9360e, BlockingEventLoop@63b1d598],
		but emission happened in [DispatchedCoroutine{Active}@71bdfe1a, Dispatchers.Default].
		Please refer to 'flow' documentation or use 'flowOn' instead
	at...
```

### flowOn operator 

아래의 예는 flow context를 변경하는 올바른 방법입니다.

```kotlin
fun flowOnOperator(): Flow<Int> = flow {
    for (i in 1..3) {
        Thread.sleep(100)
        log("Emitting $i")
        emit(i)
    }
}.flowOn(Dispatchers.Default)

fun main() = runBlocking<Unit> {
    flowOnOperator().collect { value ->
        log("Collected $value")
    }
}

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
```

```
[DefaultDispatcher-worker-1] Emitting 1
[main] Collected 1
[DefaultDispatcher-worker-1] Emitting 2
[main] Collected 2
[DefaultDispatcher-worker-1] Emitting 3
[main] Collected 3
```

여기서 관찰해야 할 또 다른 사항은 flowOn 연산자가 flow의 기본 순차 특성을 변경했다는 것입니다.
이제 수집은 하나의 coroutine("coroutine#1")에서 발생하고 방출은 수집 coroutine과 동시에 다른 스레드에서 실행 중인 다른 coroutine("coroutine#2")에서 발생합니다.

