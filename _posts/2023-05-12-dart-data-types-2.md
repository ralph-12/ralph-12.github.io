---
title: "Dart data types-2"
excerpt: "Dart의 Collections"

categories:
  - Dart
tags:
  - [Dart data types-2]

permalink: /dart/dart-data-types-2/

toc: true
toc_sticky: true

date: 2023-05-12
last_modified_at: 2023-05-12
---
<img src="/assets/images/posts_img/dart/romain-huneau-fEgt5QRI-rA-unsplash.jpg" width="600">
[Photo by Romain HUNEAU](https://unsplash.com/ko/%EC%82%AC%EC%A7%84/fEgt5QRI-rA)

### List
`List`는 데이터를 여러개 담을수 있는 자료구조입니다. 데이터의 순서가 존재하고 중복을 허용합니다.
`Dart`에서 일반적인 배열은 `List`를 뜻합니다.   


``` dart
var numbers = [1, 2, 3];
```


`List` 리터럴 마지막 아이템에 `,`를 추가하면 collection에 영향을 주지 않지만 `copy-paste` 에러를 방지할 수 있습니다.  
`,`를 추가하면 에디터가 아래와 같이 자동 정렬을 도와줍니다.  


``` dart
var isAdd = true;
var numbers = [
  1,
  2,
  3,
  4,
  6,
];
```

### Sets
`Set`은 데이터를 여러 개 담는 것은  `List`와 동일하지만 데이터의 순서가 없고 중복된 요소를 허용하지 않습니다.  


``` dart
void main() {
  var numbers = {
    1,
    2,
    3,
    5,
  };

  numbers.add(3);
  numbers.add(3);

  print(numbers);
}
```

3을 두번 추가했지만 아래와 같이 중복 되지 않음을 알 수 있습니다.   


```
{1, 2, 3, 5}

Exited.
```

### Maps
`Map`은 키와 값으로 이루어져 있습니다. 키는 `Unique`하지만 동일한 값을 여러번 사용할 수 있습니다.   


``` dart
  var player = {
    'name': 'ralph',
    'score': '99',
    'weapon': false,
  };

```
  
  
```
{name: ralph, score: 99, weapon: false}

Exited.
```
