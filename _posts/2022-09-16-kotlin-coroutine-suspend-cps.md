---
title: "Coroutine 내부 들어다보기"
excerpt: "Suspend 함수의 동작의 비밀"

categories:
  - Kotlin
tags:
  - [Kotlin]

permalink: /kotlin/kotlin-coroutine-suspend-cps/

toc: true
toc_sticky: true

date: 2022-09-16
last_modified_at: 2022-09-16
---

<img src="/assets/images/posts_img/kotlin-coroutine-suspend-cps/acton-crawford-A4xwAcZOAyo-unsplash.jpg" width="600">
[Photo by Acton Crawford](https://unsplash.com/photos/A4xwAcZOAyo)


## Suspend function 
```Coroutine```를 학습한 사람이라면 모두 들어봤을 ```Suspend function```,
기본적으로 ```Coroutine```은 순차적으로 수행하며 일시중단이 가능합니다. ```Coroutine```은 기존의 ```Callback``` 지옥에서 벗어나게 해준 고마운 존재입니다. 이런 ```coroutine```에서 ```Suspend keyword```를 붙이게 되면 일시 중단 작업이 가능한 함수로 만들수 있는데요.

```Suspend function```은 도대체 어떻게 구현이 되었길래 이게 가능할까요? 

JetBrains의 컨퍼런스 2017에서 ```Roman Elizarov```은 이렇게 말했습니다. 

<img src="/assets/images/posts_img/kotlin-coroutine-suspend-cps/kotlinConf2017_roman_elizarov.png" width="600">

[KotlinConf 2017](https://youtu.be/YrrUCSi72E8)

```Coroutine```에서 이런 동작은 마법이 아닌 일반 코드일 뿐이라는 겁니다. 

### Continuation Passing Style

더 자세히 살펴보겠습니다. ```Continuation Passing Style``` 일명 ```CPS``를 들어보셨나요? ```Suspend keyword```가 붙은 함수는 컴파일러는 ```Continuation```을 구현한 코드로 변경해 줍니다. 

```Continuation```의 인터페이스를 확인해보면 아래와 같습니다.

```kotlin
/**
 * Interface representing a continuation after a suspension point that returns a value of type `T`.
 */
@SinceKotlin("1.3")
public interface Continuation<in T> {
    /**
     * The context of the coroutine that corresponds to this continuation.
     */
    public val context: CoroutineContext

    /**
     * Resumes the execution of the corresponding coroutine passing a successful or failed [result] as the
     * return value of the last suspension point.
     */
    public fun resumeWith(result: Result<T>)
}
``` 

```context``` 객체와 ```resumeWith``` 함수가 보입니다. ```CPS```에서는 ```Continuation```를 계속해서 전달하면서 
```suspend```로 중단된 부분을 다시 ```resume```이 가능하게 도와줍니다. 

이제 JVM이 ```Suspend function```를 분석하는 방식을 알아봅시다. 


