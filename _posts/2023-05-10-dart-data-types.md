---
title: "Dart data types"
excerpt: "Dart data 기본 types 톺아보기"

categories:
  - Dart
tags:
  - [Dart data types]

permalink: /dart/dart-data-types/

toc: true
toc_sticky: true

date: 2023-05-10
last_modified_at: 2023-05-12
---
<img src="/assets/images/posts_img/dart/mick-haupt-3UbsiRcrFV4-unsplash.jpg" width="600">
[Photo by Mick Haupt](https://unsplash.com/ko/%EC%82%AC%EC%A7%84/3UbsiRcrFV4)  

## Dart data types
`Dart`의 모든 변수는 `object`이며 `object`는 `class`의 `instance`입니다.  
또한 모든 타입은 레퍼런스 타입입니다. 

### Number
`int`와 `double`이 존재합니다. 

#### int

``` dart 
var x = 1;
var hex = 0xDEADBEEF;
```

#### double 

``` dart
var y = 1.1;
var exponents = 1.42e5;
```

#### num 
`num`으로 선언해서 사용할 수 있습니다. `num`으로 선언시 `int`와 `double` 모두 사용 가능합니다. 

```dart 
num x = 1;
x = 1.1;
x += 2.5;
```

또한 아래와 같이 정수 리터럴은 `double`형으로 자동 컨버팅 됩니다.

``` dart 
double z = 1;
```

형변환시 다음과 같이 가능합니다. 
``` dart 
// String to int 
var one = int.parse('1');

// string to doulbe
var onePointOne = double.parse(1.1);

// int to String 
String oneAsString = 1.toString();

// double to String 
String piAsString = 3.14159.toStringAsFixed(2);
```

`int`는 bit 연산도 가능합니다. 
``` dart 
assert((3 << 1) == 6); // 0011 << 1 == 0110
assert((3 | 4) == 7); // 0011 | 0100 == 0111
assert((3 & 4) == 0); // 0011 & 0100 == 0000
```

### Strings
`String`은 ', " 모두 사용 가능합니다. 

``` dart 
var name = 'ralral-phph';
String name2 = "ralral-phph";
```

#### Strings interpolation
`kotlin` 처럼 `$`를 붙여 사용 가능합니다. 

``` dart
var name = 'ralph';
var age = 30;
var greeting = 'Hello everyone, my name is $name,'
  ' nice to meet you and I\`m ${age + 2}';
print(greeting);  
```

#### multi-line string 
multi-line string은 `'''`를 붙이거나 `''`로 가능합니다.

``` dart
var multi = '''
  multi-line string
  multi-line string
  multi-line string
  ''';
print(multi);

var multi2 = 'multi2-line string\n'
      'multi-line string\n'
      'multi-line string\n';
print(multi2);
```

### boolean
`true`, `false` 값을 가지며 컴파일 타임 상수입니다.

``` dart
bool isOk = true;
  if (isOk) {
    print(isOk);
  }
```
