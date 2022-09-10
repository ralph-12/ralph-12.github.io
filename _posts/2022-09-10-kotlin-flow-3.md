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

Kotlin 표준 라이브러리의 Sequence.zip 확장 함수와 마찬가지로 flow에는 두 flow의 해당 값을 결합하는 ```zip``` 연산자가 있습니다.

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
