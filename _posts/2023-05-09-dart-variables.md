---
title: "Dart variables"
excerpt: "Dart variables"

categories:
  - Dart
tags:
  - [Dart variables]

permalink: /dart/dart-variables/

toc: true
toc_sticky: true

date: 2023-05-09
last_modified_at: 2023-05-09
---
<img src="/assets/images/posts_img/dart/miikka-luotio-i3WlrO7oAHA-unsplash.jpg" width="600">
[Photo by Miikka Luotio](https://unsplash.com/ko/%EC%82%AC%EC%A7%84/i3WlrO7oAHA)  
  
  


## Dart variables
최근 `Flutter`를 이용해 개발을 해야해서 차근차근 `Dart`를 공부하고자 합니다.   
   
### var
 
 `Dart`는 아래와 같이 변수를 선언하고 초기화 할수 있습니다. 
 
 ``` dart
 var name = 'ralral-phph' // dart에서 문자열은 (',") 모두 사용 가능합니다.
 String name2 = 'ralral-phph' // 다음과 같이 변수의 유형을 지정해 선언할 수 있습니다. 
 
 ```
 
구글은 변수의 유형을 지정해 주는 것보다 `var` 키워드를 사용하는 것을 권장하고 있습니다. 
  
  

### dynamic

위에서 살펴본 `var`은 한번 추론되면 다른 타입을 저장할 수 없습니다. 

``` dart
var name = 'ralral-phph'
name = 1234 // error
```

`dynamic` 타입은 동적으로 타입을 변경할수있습니다. 

```dart
dynamic name = 'ralral-phph'
name = 1234 
```
  
  
### Late variables

`late` 변수는 다음과 같이 사용할 수 있습니다. 
- 선언 후에 초기화되는 `non-nullable` 변수를 선언합니다.
- 지연 초기화가 필요할 때 사용합니다. 

``` dart 
late final String name; 
// do something... fetch api 
name = temp;
```

### Final and const
`final`과 `const` 모두 값을 변경할 수 없는 변수를 만들수 있지만 아래와 같은 차이점이 있습니다.

``` dart
final String ip = '110.44.44444.'; 
final DateTime now = DateTime.now(); // run time에서 결정되는 값도 설정할수 있어요
const API_KEY = 'DFDFDFDADFASD'; // compile time에 값을 결정합니다.
```
  
  
### Null safety
`Null safety`는 null로 인한 run-time 에러를 방지하고 edit-time에서 에러를 내어 안전성을 확보합니다.  

`Dart`는 기본적으로 변수에 null을 할당 할수 없습니다. 그러나 `?`를 붙여서 예외적으로 null을 허용할 수 있습니다.

``` dart
String? name = 'ralral-phph';
name = null;

if (name != null) { // null check
  name.isNotEmpty;
}

name?.isNotEmpty; // 이와 같이 null check도 가능합니다.
```
