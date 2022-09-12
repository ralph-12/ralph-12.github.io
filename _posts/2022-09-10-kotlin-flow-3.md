---
title: "Asynchronous Flow 알아보기 3"
excerpt: "가깝고 먼 Asynchronous Flow 알아보기 3편"

categories:
  - Kotlin
tags:
  - [Kotlin]

permalink: /kotlin/asynchronous-flow-3/

toc: true
toc_sticky: true

date: 2022-09-10
last_modified_at: 2022-09-11
---

### Flow exceptions

코드에서 예외가 발생하면 flow 수집이 예외와 함께 완료될 수 있습니다. 
예외를 처리하는 방법에는 여러가지가 있습니다.


#### Collector try and catch 
kotlin의 try/catch 블록을 사용하여 예외를 처리할 수 있습니다.

```kotlin
fun collectorTryNCatch(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i) // emit next value
    }
}

fun main() = runBlocking {
    try {
        collectorTryNCatch().collect { value ->
            println(value)
            check(value <= 1) { "Collected $value" }
        }
    } catch (e: Throwable) {
        println("Exception $e")
    }
}
```

```
Emitting 1
1
Emitting 2
2
Exception java.lang.IllegalStateException: Collected 2
```

#### Everything is Caught

위의 예제는 ```emitter```에서 발생하는 어떤 exception도 포착합니다.
```map```을 통해 방출되는 값을 strings로 변경해 봅시다. 

```kotlin
fun everythingIsCaught(): Flow<String> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}
    .map { value ->
        check(value <= 1) { "Crashed on $value" }
        "string $value"
    }

fun main() = runBlocking<Unit> {
    try {
        everythingIsCaught().collect{ value ->
            println(value)
        }
    } catch (e: Throwable) {
        println("Exception $e")
    }
}
```
아래와 같이 출력합니다. 
```
Emitting 1
string 1
Emitting 2
Exception java.lang.IllegalStateException: Crashed on 2
```

### Exception transparency

예외 처리 동작을 어떻게 캡슐화 할까요 

```emitter```는 이 예외 투명도를 유지하고 예외 처리의 캡슐화를 허용하는 ```catch``` 연산자를 사용할 수 있습니다. 
```catch``` 연산자의 본문은 예외를 분석하고 포착된 예외에 따라 다양한 방식으로 대응할 수 있습니다.

* 예외는 ```throw```를 사용하여 다시 ```throw```될 수 있습니다.
* catch의 body에서 예외를 방출할 수 있습니다.
* 예외는 다른 코드에서 무시, 로그, 처리될 수 있습니다.

예를 들어 예외를 잡아낼 때 텍스트를 내보냅니다.

```kotlin
simple()
    .catch { e -> emit("Caught $e") } // emit on exception
    .collect { value -> println(value) }
```

#### Transparent catch

catch 중간 연산자는 업스트림 예외만 포착합니다. 

```kotlin
fun transparentCatch(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    transparentCatch()
        .catch { e -> println("Catch $e") }
        .collect { value ->
            check(value <= 1) { "Collected $value"}
            println(value)
        }
}
```

catch 연산자가 있음에도 "Cath ..." 메시지가 출력되지 않습니다. 
```
Emitting 1
1
Emitting 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
```

#### Catching declaratively
```collect``` 연산자를 ```onEach```로 옮겨서 그것을 ```catch``` 연산자 앞에 두면 모든 예외를 처리하는 부분과 ```catch```연산자의 동작 부분을 결합할 수 있습니다. 이 flow의 수집은 매개변수 없이 ```collect()```를 호출하여 작동 시켜야 합니다.

```kotlin
fun catching(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    catching()
        .onEach { value ->
            check(value <= 1) { "Collected $value"}
            println(value)
        }
        .catch { e -> println("Catch $e") }
        .collect()
}
```

이제 "Catch ..." 메시지가 출력되어 명시적으로 try/catch 블록을 사용하지 않고도 모든 예외를 catch할 수 있습니다.
```
Emitting 1
1
Emitting 2
Catch java.lang.IllegalStateException: Collected 2
```

### Flow completion
flow 수집이 완료되면 (일반적으로 또는 예외적으로) 작업을 실행해야 할 수 있습니다. 명령형과 선언형의 두 가지 방법으로 수행할 수 있습니다.

#### Imperative finally block
try/catch 외에도 finally 블록을 사용하여 수집 완료 시 작업을 실행할 수도 있습니다.

```kotlin
fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } finally {
        println("Done")
    }
}
```

이 코드는 1부터 3까지 차례대로 출력 후 finally {...} 문의 "Done"을 출력합니다. 
```
1
2
3
Done
```

#### Declarative handling

선언적 접근 방식에서는 flow가 완전히 collect 되었을 때 호출되는 ```onCompletion```이라는 중간 연산자를 가지고 있습니다.

이전 예제를 ```onCompletion``` 연산자를 사용하여 다시 작성하겠습니다.

```kotlin
simple()
    .onCompletion { println("Done") }
    .collect { value -> println(value) }
```


```kotlin
fun declarativeSimple(): Flow<Int> = flow {
    emit(1)
    throw RuntimeException()
}

fun main() = runBlocking<Unit> {
    declarativeSimple()
        .onCompletion { cause -> if (cause != null) println("Flow completed exceptionally") }
        .catch { cause -> println("Caught exception") }
        .collect { value -> println(value) }
}
```
```onCompletion```의 이점은 flow 수집이 정상적으로 완료되었는지 또는 예외적으로 완료되었는지 여부를 결정하는데 사용하는 람다의 ```nullble```한 ```Throwable``` 매개변수입니다.
예제는 숫자 1을 방출한 후 예외를 발생시킵니다. 
```
1
Flow completed exceptionally
Caught exception
```

#### Successful completion

```catch``` 연산자와의 또 다른 차이점은 onCompletion이 모든 예외를 확인하고 업스트림 flow가 성공적으로 완료된 경우에만(취소 또는 실패 없이) null 예외를 수신한다는 것입니다.

```kotlin
fun successfulCompletion(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    successfulCompletion()
        .onCompletion { cause -> println("Flow completed with $cause")  }
        .collect { value ->
            check(value <= 1) { "Collected $value"}
            println(value)
        }
}
```

다운스트림 예외로 인해 flow가 중단되었기 때문에 완료 원인이 null이 아님을 알 수 있습니다.
```
1
Flow completed with java.lang.IllegalStateException: Collected 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
at ...
```

### Imperative versus declarative
 flow를 수집하는 방법과 명령적 방법과 선언적 방법으로 완료와 예외를 처리하는 방법을 알았습니다.
두 가지 방법 중 코드 스타일이나 선호도에 따라 선택하여 사용하면 됩니다.

### Launching flow 
flow를 사용하여 일부 소스에서 오는 비동기 이벤트를 처리하는 것은 쉽습니다. 이 경우에 들어오는 이벤트에 대한 방법으로 
코드를 작성하고 추가 작업을 계속하는 ```addEventListener```함수의 유사체가 필요합니다.

```onEach```가 이 역할을 수행할 수 있지만 flow를 수집하려면 terminal 연산자도 필요합니다. 그렇지 않으면 ```onEach```를 호출하는 것만으로는 효과가 없습니다.

```onEach``` 다음에 collet terminal 연산자를 사용하면 그 뒤의 코드는 flow가 수집될 때까지 기다립니다.

```kotlin
// Imitate a flow of events
fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event : $event") }
        .collect() // <--- Collecting the flow
    println("Done")
}
```
아래와 같이 출력합니다. 
```
Event : 1
Event : 2
Event : 3
Done
```

launchIn terminal 연산자는 편리하게 사용할 수 있습니다.
collect를 launchIn으로 대체하여 별도의 코루틴에서 flow 컬렉션을 시작할 수 있으므로 추가 코드 실행이 즉시 계속됩니다.

```kotlin
fun launchInEvents(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    launchInEvents()
        .onEach { event -> println("Event: $event") }
        .launchIn(this) // <--- Launching the flow in a separate coroutine
    println("Done")
}
```

```
Done
Event: 1
Event: 2
Event: 3
```

```launchIn```에 대한 필수 매개변수는 flow를 수집할 ```coroutine```의 범위 ```CoroutineScope```를 지정해야 합니다.
위의 예에서 이 범위는 ```runBlocking``` 코루틴 빌더에서 가져오므로 flow가 실행되는 동안 ```runBlocking``` 범위는 자식 ```coroutine```이 완료될 때까지 대기하고 주 함수가 이 예를 반환하고 종료하지 않도록 합니다.

실제 어플리케이션에서 범위는 수명이 제한된 엔티티로 부터 가져옵니다. 이 엔티티의 라이프사이클이 종료되는 즉시 해당 범위가 취소되고 해당 flow의 수집이 취소됩니다.

```launchIn```은 또한 전체 범위를 취소하거나 조인하지 않고 해당 flow 수집 ```coroutine```를 취소하는 데 사용할 수 있는 ```Job```을 반환합니다.
 
