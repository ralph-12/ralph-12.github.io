---
title: "Hilt & Dagger Annotations"
excerpt: "다양한 Hilt & Dagger Annotation 알아보기"

categories:
  - Android
tags:
  - [Android]

permalink: /android/hilt-dagger-annotations/

toc: true
toc_sticky: true

date: 2022-08-25
last_modified_at: 2022-08-25
---

## Hilt & Dagger Annotations 알아보기 
<img src="/assets/images/posts_img/hilt-dagger-annotations/markus-spiske-DnBtFBnqlRc-unsplash.jpg" width="800">
(이미지 출처 : https://unsplash.com/photos/DnBtFBnqlRc)

### Hilt 가 무엇일까요
* Hilt는 Android 전용 의존성 주입(Dependency Injection) 라이브러리 입니다.
* Hilt는 의존성을 주입하고 수명주기를 자동적으로 관리해주고 있습니다.

### DI? 의존성 주입?
객체가 필요할때 마다 생성을 하면 객체간의 의존성이 커지게 됩니다.
그렇기 때문에 객체를 직접 생성하는 것이 아닌 외부로 부터 필요한 객체를 받아 사용하게 하는 것을
의존성 주입이라고 합니다. 

### DI의 장점
그렇다면 DI를 통해 얻을 수 있는 이점이 무엇일까요?
 * Unit Test가 용이해 집니다.
 * 객체간의 의존성을 줄이거나 없애고 코드의 재활용성을 높여줍니다.
 * 객체 간의 결합도를 낮추면서 유연한 코드를 개발자는 작성할 수 있게됩니다 😃

### Android에서 DI 선택지 Dagger vs Koin vs Hilt 
Android 개발자라면 누구나 들어봤을 DI Framework 들이 있습니다. 
바로 Dagger, Koin, Hilt 인데요. 대략적인 특징을 알아보겠습니다.

1) [Dagger](https://github.com/google/dagger)

```
구글에서 제공하는 라이브러리인 Dagger는 처음엔 Android보다 Java Spring 등의 환경에서
많이 활용되었습니다. 하지만 많은 개발자가 Android 환경에서 많이 사용하다 보니 Dagger-Android도
제공하기 시작했습니다. 그러나 Component 관계 설정, Scope의 개념이라든지 개발자가 적용하기 위해서
많은 학습비용을 요구한다는 단점이 있습니다. 
```

2) [Koin](https://insert-koin.io/)

```
Koin은 코틀린 환경에서 사용할 수 있으며 Kotlin DSL로 구성되어 있습니다.
위의 Dagger에 비하면 사용 방법이 쉽고 도입이 빠르다는 장점이 있습니다.
그러나 Runtime에서 의존성을 주입하고 에러 검출 시점이 Runtime에서 발생합니다.
또한 리플랙션을 이용해서 성능이 떨어집니다.
최근 많은 상용 프로젝트에서도 Koin을 사용하는 것을 볼 수 있습니다.
```

3) [Hilt](https://developer.android.com/training/dependency-injection/hilt-android?hl=ko)

```
Hilt는 Dagger를 기반으로 구성되어 있지만 Dagger를 쉽게 사용하게 고안되어 그렇게 높은 학습비용을 요구하지 않습니다.
Compile 단계에서 의존성을 주입하고 에러 검출 또한 Copile 단계에서 발생합니다.
Hilt는 프로젝트의 모든 Android 클래스에 컨테이너를 제공하고 수명 주기를 자동으로 관리합니다.
Android developers에서도 가이드를 제공하는 만큼 Android 개발자라면 사용 안 할 이유가 없습니다.
많은 사용자 수를 가지고 있는 서비스 기업들도 Koin에서 Hilt로 이전 하고 있습니다.
```

## Hilt Annotation 알아보기 

### @HiltAndroidApp

Hilt 코드 생성을 시작합니다. Application 클래스에 주석을 달아야 합니다.

```kotlin
@HiltAndroidApp
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
    }
}
```

### @AndroidEntryPoint

객체를 주입할 대상에게 어노테이션을 붙여 member injection를 수행합니다. 
Activity, Fragment, View, Service, BroadcastReceiver에 사용 가능합니다. 

```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```
