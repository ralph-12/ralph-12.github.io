---
title: "Asynchronous Flow 알아보기 1"
excerpt: "가깝고 먼 Asynchronous Flow 알아보기 1편"

categories:
  - Kotlin
tags:
  - [Kotlin]

permalink: /kotlin/asynchronous-flow-1/

toc: true
toc_sticky: true

date: 2022-09-05
last_modified_at: 2022-09-09
---

## Asynchronous Flow
suspand 함수는 단일 값을 비동기적으로 반환하지만 어떻게 여러개의 값을 반환할 수 있을까요?
Kotlin Flows가 이를 해결해 줍니다. 

이 포스트는 [kotlinlang.org](https://kotlinlang.org/docs/flow.html) 의 flow 예제에 관한 글입니다.


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
이 코드는 메인 스레드를 차단하지 않고 각 숫자를 인쇄하기 전에 기다린 후 메인 스레드에서 실행되는 별도의 coroutine에서
100ms마다 "I'm not locked"를 인쇄하여 확인합니다.

```
I'm not blocked 1
1
I'm not blocked 2
2
I'm not blocked 3
3
```
  
### Flows are cold
시퀸스와 유사한 콜드 스크림입니다. flow 빌더 내부의 코드는 flow가 수집 되기 전 실행되지 않습니다.

```kotlin
  
fun flowsAreCold(): Flow<Int> = flow {
    println("Flow started")
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    println("Calling simple function...")
    val flow = flowsAreCold()
    println("Calling collect...")
    flow.collect { value -> println(value)}
    println("Calling collect again...")
    flow.collect { value -> println(value)}
}
```
  
```
Calling simple function...
Calling collect...
Flow started
1
2
3
Calling collect again...
Flow started
1
2
3   
```

이는 flowsAreColde 함수에 suspend로 표시되지 않는 주요 이유 입니다.
그 자체로 flowsAreColde() 함수의 호출은 빠르게 반환되고 아무것도 기다지리 않습니다.
flow는 수집될 때마다 시작되므로 다시 호출될 때 "Flow started"가 출력됩니다.


### Flow cancellation basics
Flow는 coroutine의 일반적인 협력 취소를 따릅니다. 평소와 같이 취소 가능한 일시 중단 기능(예: delay)에서 흐름이 일시 중단되면 흐름 수집이 취소될 수 있습니다. 다음 예제는 withTimeoutOrNull 블록에서 실행될 때 흐름이 시간 초과 시 취소되고 코드 실행을 중지하는 방법을 보여줍니다.
 
```kotlin
  
fun flowCancellationBasics(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    withTimeoutOrNull(250) {
        flowCancellationBasics().collect { value -> println(value) }
    }
    println("Done")
}
```

단순 함수의 flow에서 두 개의 숫자만 내보내고 다음 출력을 생성하는 방법에 주목하세요.
  
```
Emitting 1
1
Emitting 2
2
Done
```

### Flow builders
이전 예제의 flow { ... } 빌더가 가장 기본적인 것입니다. flow를 더 쉽게 선언할 수 있는 다른 빌더가 있습니다.

* 고정된 값 집합을 내보내는 flow를 정의하는 flowOf 빌더입니다.
* .asFlow() 확장 함수를 사용하여 다양한 컬렉션과 시퀀스를 flow로 변환할 수 있습니다.

```kotlin

// Convert an integer range to a flow
fun main() = runBlocking<Unit> {
    (1..3).asFlow().collect { value -> println(value) }
}
```

```
1
2
3
```

### Intermediate flow operators
collections 및 sequences와 마찬가지로 연산자를 사용하여 flow를 변환할 수 있습니다. 중간 연산자는 업스트림 flow에 적용되고 다운스트림 flow를 반환합니다. 이 연산자는 flow가 그렇듯이 차갑습니다. 빠르게 작동하여 새로운 변환된 flow의 정의를 반환합니다.

기본 연산자는 map와 filter와 같은 익숙한 이름을 가지고 있습니다. sequences와 중요한 차이점은 이러한 연산자 내부의 코드 블록이 일시 중단 함수를 호출할 수 있습니다.
 
예를 들어 요청을 수행하는 것이 일시 중단 기능에 의해 구현되는 장기 실행 작업인 경우에도 들어오는 요청의 flow를 map 연산자를 사용하여 결과에 매핑할 수 있습니다.

```kotlin

suspend fun performRequest(request: Int): String {
    delay(1000)
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow()
        .map { request -> performRequest(request) }
        .collect { response -> println(response)}
}
```

```
response 1
response 2
response 3
```

### Transform operator

flow 변환 연산자 중 가장 일반적인 것은 transform이라고 합니다. map 및 filter와 같은 간단한 변환을 모방하고 더 복잡한 변환을 구현하는 데 사용할 수 있습니다. transform 연산자를 사용하여 임의의 값을 임의의 횟수만큼 방출할 수 있습니다.

예를 들어, 변환을 사용하여 장기 실행 비동기 요청을 수행하기 전에 문자열을 내보내고 응답을 따라갈 수 있습니다.

```kotlin

suspend fun transformOperator() {

    (1..3).asFlow() // a flow requests
        .transform { request ->
            emit("Making request $request")
            emit(performRequest(request))
        }
        .collect { response -> println(response) }
}

fun main() = runBlocking{
    transformOperator()
}
```

```
Making request 1
response 1
Making request 2
response 2
Making request 3
response 3
```

### Size-limiting operators

take와 같은 크기 제한 중간 연산자는 해당 제한에 도달하면 flow 실행을 취소합니다. coroutine의 취소는 항상 예외를 throw하여 수행됩니다.
모든 리소스 관리 기능(예: try { ... } finally { ... } 블록)은 취소 시 정상적으로 작동합니다.

```kotlin
fun numbers(): Flow<Int> = flow<Int> {
    try {
        emit(1)
        emit(2)
        println("This line will not execute")
        emit(3)
    } finally {
        println("Finally in numbers")
    }
}

fun main() = runBlocking {
    numbers()
        .take(2)
        .collect { value -> println(value) }
}
```

이 코드의 출력은 두 번째 숫자를 emit한 후 numbers() 함수에서 flow { ... } 본문의 실행이 중지되었음을 명확하게 보여줍니다.

```
1
2
Finally in numbers
```

### Terminal flow operators

terminal flow 연산자는 flow 컬렉션을 시작하는 함수를 일시 중단합니다. collect 연산자는 가장 기본적인 연산자이지만 더 쉽게 만들 수 있는 다른 터미널 연산자가 있습니다.

* toList 및 toSet과 같은 다양한 컬렉션으로의 변환
* 첫 번째 값을 가져오고 flow가 단일 값을 내보내도록 하는 연산자입니다.
* reduce 및 fold를 사용하여 flow를 값으로 줄여줍니다.
* reduce는 주어진 연산을 누적시키면서 최종값을 반환합니다.
* fold는 초기 값을 입력받아 주어진 operation을 이용하여 누적시키면서 최종값을 반환합니다. 
  
```kotlin
fun main() = runBlocking {
    val sum = (1..5).asFlow()
        .map { it * it } // squares of numbers from 1 to 5
        .reduce { a, b -> a + b }

    println(sum)
}
```

```
55
```

### Flows are sequential
여러 flow에서 작동하는 특수 연산자를 사용하지 않는 한 flow의 각 개별 collection은 순차적으로 수행됩니다.
collection은 terminal 연산자를 호출하는 coroutine에서 직접 작동합니다. 기본적으로 새로운 coroutine에서 실행되지 않습니다.
내보낸 각 값은 모든 중간 운영자가 업스트림에서 다운스트림으로 처리한 후 terminal 운영자에게 전달됩니다.


```kotlin
fun main() = runBlocking<Unit> {
    (1..5).asFlow()
        .filter {
            println("Filter $it")
            it % 2 ==0
        }
        .map {
            println("Map $it")
            "string $it"
        }.collect {
            println("Collect $it")
        }
}
```

예제에 따르면 짝수 정수를 filter하고 문자열에 map하고 있습니다. 

```
Filter 1
Filter 2
Map 2
Collect string 2
Filter 3
Filter 4
Map 4
Collect string 4
Filter 5
```




