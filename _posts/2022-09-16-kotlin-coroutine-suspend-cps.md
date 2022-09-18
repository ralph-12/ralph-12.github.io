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

```Coroutine```를 학습한 사람이라면 모두 들어봤을 ```Suspend function```, 기본적으로 ```Coroutine```은 순차적으로 수행하며 일시중단이 가능합니다. 
```Coroutine```은 기존의 ```Callback``` 지옥에서 벗어나게 해준 고마운 존재입니다. 
이런 ```Coroutine```에 ```Suspend keyword```를 붙이게 되면 일시 중단 작업이 가능한 함수로 만들수 있는데요.


```Suspend function```은 내부적으로 어떻게 구현이 되었길래 이게 가능할까요? 


JetBrains의 컨퍼런스 2017에서 ```Roman Elizarov```은 이렇게 말했습니다. 

<img src="/assets/images/posts_img/kotlin-coroutine-suspend-cps/kotlinConf2017_roman_elizarov.png" width="600">

[KotlinConf 2017](https://youtu.be/YrrUCSi72E8)

```
There is no magic 
```

```Coroutine```에서 이런 동작은 마법이 아닌 일반 코드일 뿐이라는 겁니다.

더 자세히 살펴보도록 하겠습니다. 

### Continuation Passing Style

```Suspend keyword```가 붙은 함수를 컴파일러는 ```Continuation```을 구현한 코드로 변경해 줍니다. 
```Continuation Passing Style``` 일명 ```CPS```로 변환해주는 것입니다. 


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


아래 간단한 코드를 작성해 보겠습니다. 

```kotlin
fun main() = runBlocking<Unit> {
    GlobalScope.launch {
        val profile = fetchUser()
        val convertUser = converterUser(profile)
        fetchUserSummery(convertUser)
    }
}

suspend fun fetchUser() = UserData(199212,  "테스트", 30)

fun converterUser(userData: UserData): String = "이름은 ${userData.name}, {$userData.age}살 입니다."

suspend fun fetchUserSummery(description: String): UserSummery = UserSummery(description)
```

User 정보를 가져와 User 요약으로 변환해주는 코드가 있다고 해보겠습니다. 

이를 Bytecode로 변환해보겠습니다.

```java
  @Nullable
   public static final Object fetchUser(@NotNull Continuation $completion) {
      return new UserData(199212, "테스트", 30);
   }

   @Nullable
   public static final Object converterUser(@NotNull UserData userData, @NotNull Continuation $completion) {
      return "이름은 " + userData.getName() + ", {" + userData + ".age}살 입니다.";
   }

   @NotNull
   public static final UserSummery fetchUserSummery(@NotNull String description) {
      Intrinsics.checkNotNullParameter(description, "description");
      return new UserSummery(description);
   }
   
```
일반함수로 변환되었으며 ```Continuation Passing Style```로 변환 된 것으로 확인 할 수 있습니다. 


### Labels

```Coroutine```의 빌더 구문에서 순차적으로 작성한 코드들은 디컴파일할 때 Labels이 생성되게 됩니다.
아래의  ```switch```문이 보이시나요?

```java
switch(this.label) {
  case 0:
    ResultKt.throwOnFailure($result);
    this.label = 1;
    var10000 = Example_cpsKt.fetchUser(this);
    if (var10000 == var4) {
    return var4;
    }
    break;
  case 1:
    ResultKt.throwOnFailure($result);
    var10000 = $result;
    break;
  case 2:
    ResultKt.throwOnFailure($result);
    var10000 = $result;
    break label17;
  default:
    throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
  }
...
```

### 마무리 
정리하자면 ```Suspend function```은 내부적으로 Lable를 매기고 ```switch case```문 형태로 만들어져서 
```CPS Callback Interface``` 형태로 이루어짐을 알수 있습니다. 
