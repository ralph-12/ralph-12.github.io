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

### 명시적인 ```Job```에 대해 취소 하기

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
suspension point로 delay를 800L 만큼 주었기 때문에 delay가 1000L인 job1만 출력 못한 것을 
알 수 있습니다. 

```
launch1: main
launch2: main
2!
launch3: main
3!
4!
runBlocking: main
5!
```

### 취소가 불가능한 Job

위의 예제는 모두 ```runBlocking```을 통해 Main thread에서 진행한 작업입니다. 
Job을 메인스레드가 아닌 다른 Dispatcher로 동작하게 해보고 취소 해보겠습니다.

```kotlin
suspend fun doCount() = coroutineScope {

    val job1 = launch(Dispatchers.Default) {
        println("doCount : ${Thread.currentThread().name}")
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while( i <= 10) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }

    delay(200L)
    job1.cancel()
    println("doCount Done!")
}

fun main() = runBlocking {
    println("main : ${Thread.currentThread().name}")
    doCount()
}
```

분명 cancle()를 했음에도 취소가 되지 않았음을 확인 할 수 있습니다. 

```
main : main
doCount : DefaultDispatcher-worker-1
1
2
doCount Done!
3
4
5
6
7
8
9
10
```

###Cancel(), join()

위의 예제에 join()를 추가해 어떻게 동작하는지 확인해 보겠습니다. 

```kotlin
suspend fun cancelAndJoinExample() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }

    delay(200L)
    job1.cancel()
    job1.join()
    println("doCount Done!")
}

fun main() = runBlocking {
    cancelAndJoinExample()
}
```

우리가 의도한대로 suspend 함수가 실제로 끝날때 까지 기달린 후에 출력한 것을 확인 할 수 있습니다. 

```
1
2
3
4
5
6
7
8
9
10
doCount Done!
```

위와 같은 상황이 빈번하게 발생하기 때문에 Coroutine에서 cancelAndJoin()를 제공해 줍니다.

```kotlin
job1.cancelAndJoin()
```
