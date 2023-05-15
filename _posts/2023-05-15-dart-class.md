---
title: "Dart class"
excerpt: "Dart의 class"

categories:
  - Dart
tags:
  - [Dart class]

permalink: /dart/dart-class/

toc: true
toc_sticky: true

date: 2023-05-15
last_modified_at: 2023-05-15
---
<img src="/assets/images/posts_img/dart/rezvani-IIDZ77VDVQE-unsplash.jpg" width="600">
[Photo by Rezvani](https://unsplash.com/ko/%EC%82%AC%EC%A7%84/IIDZ77VDVQE)

### Class
`class`를 만들고 `class`의 멤버들은 다음과 같이 접근할 수 있습니다. 

``` dart
class Person {
  String name = 'ralph';
  int age = 20;
  final String country = 'seoul, korea';
  
  void sayHello() {
    print('Hi my name is $name');
  }
}

void main() {
  var person = Person();
  var name = person.name;
  print(name);
  player.name = 'Too';
  print(player.name);
  
  person.sayHello();
}
```

```
ralph
Too
Hi my name is Too

Exited.
```

#### constructors
기본적으로 생성자는 `class`와 동일한 이름을 가진 함수를 생성하여 선언합니다.

``` dart
Person(this.name, this.age);
```

#### Named constructors

``` dart
...
Person.origin(String name, int age)
      : name = 'ralph',
        age = 20;

// use
var person = Person.origin('tete', 30);
print(person.name);
```

```
tete

Exited.
```

다음과 같이도 사용 가능합니다.  

``` dart
...
Person.createNewPerson({
  required String name,
  required int age,
})  : this.name = name,
      this.age = age;
        
// use
var newPerson = Person.createNewPerson(
  name: 'haley',
  age: 25,
);

newPerson.sayHello();
```

```
Hi my name is haley

Exited.
```

#### Cascade Notation 

Casecade notation `..`, `?..` 을 사용해서 반복되는 이름을 생략 가능합니다.   

``` dart
// as is
var newPerson = Person.createNewPerson(
  name: 'haley',
  age: 25,
);
newPerson.name = 'ralph';
newPerson.age = 30;

// to be
var newPerson = Person.createNewPerson(
  name: 'haley',
  age: 25,
)
  ..name = 'ralph'
  ..age = 30;
```
  
#### Enum
다른 여러 언어들 처럼 열거형 클래스인 `Enum`도 지원합니다. 

``` dart
enum Team { red, blue }

class Player {
  String name;
  int age;
  Team team;
  
  Player({
    required this.name,
    required this.age,
    required this.team,
  });

...
var ralph = Player(
    name: 'ralph',
    age: 32,
    team: Team.red, // enum 
  );
```

#### Abstract class  
아래와 같이 추상 클래스를 정의할 수 있습니다. 

``` dart 
abstract class Human {
  void walk();
}

class Player extends Human {
  String name;
  int age, xp;
  Team team;


  void walk() {
    print('im walking');
  }

  Player({
    required this.name,
    required this.age,
    required this.xp,
    required this.team,
  });
}

class Coach extends Human {
  @override
  void walk() {
    print('the coach is walking');
  }
}

```

#### Inheritance
`Dart`는 객체지향 프로그래밍이기 때문에 상속도 가능합니다. 부모 클래스는 `Super class`이며 상속 받는 자식 클래스는 `Sub class`가 됩니다. 

``` dart
enum Team { blue, red} // enum

class Human { // super class 
  final String name;

  Human({required this.name});

  void sayHello() {
    print("Hi my name is $name");
  }
}

class Player extends Human { // sub class 
  final Team team;

  Player({
    required this.team,
    required String name,
  }) : super(name : name); // super class 접근하기

  
  @override
  void sayHello() {
    super.sayHello();
    print('and I play for ${team}');
  }
}

void main() {
  var player = Player(team: Team.red, name: 'ralph',);

  player.sayHello();
}
```

```
Hi my name is ralph
and I play for Team.red

Exited.
```

#### Mixins
`Mixin`은 여러 클래스 계층 구조에서 클래스의 코드를 재사용 하게 해줍니다.

``` dart
class Strong {
  final double strengthLevel = 1000.99;
}

class QuickRunner {
  void runQuick() {
    print('run!');
  }
}

class Tall {
  final double height = 1.99;
}

enum Team {red, blue}

class Player with Strong, QuickRunner, Tall {
  final Team team;
  
  Player (this.team);
}

class Horse with Strong, QuickRunner, Tall {

}


void main() {
  var player = Player(Team.blue);
  player.runQuick();
}

```

```
run!

Exited.
```

