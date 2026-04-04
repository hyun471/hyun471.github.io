---
title: "Collections"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 Collections 살펴보기

## Collections이란?

- 여러 개의 변수 값을 한꺼번에 담는 자료 구조
## Collections 종류

### List

```dart
List<int> numbers = [1, 2, 3, 4, 5];
```

- 순서가 있는 값들 이 묶인 형태 즉, 배열의 형태
- 필수 구성 요소 : List<타입> + 변수 = [요소]
  - 모든 요소의 타입은 같아야 한다.
- Dart 문법의 타입 추론을 활용하면 List없이 만들 수 있다.
```dart
var numbers = [1, 2, 3, 4, 5];             // List 요소의 변동이 있는 경우
const names = ['Alice', 'Bob', 'Henry'];   // List 요소의 변동이 없는 경우
```

  - 요소에 변동이 있는 경우 var로 선언이 가능, 변동이 없는 경우 final/const로 선언이 가능
- Index를 통해 각 요소에 접근이 가능하다.
  - index는 0부터 시작하는 정수이고, 첫 번째 Index는 0이다.
```dart
var numbers = [1, 2, 3, 4, 5];
print(numbers[3]); // 4

numbers[3] = 2;
print(numbers[3]); // 2
```

- List에서 자주 사용되는 옵션
- List에서 자주 사용되는 옵션 예시 코드
```dart

void main() {
  List<int> numbers = [1, 2, 3];

  print(numbers.length);      // 3
  print(numbers.isEmpty);     // false
  print(numbers.indexOf(2));  // 1

  numbers.add(4);
  print(numbers);             // [1, 2, 3, 4]

  numbers.addAll([5, 6]);
  print(numbers);             // [1, 2, 3, 4, 5, 6]

  numbers.remove(3);
  print(numbers);             // [1, 2, 4, 5, 6]

  numbers.removeAt(0);
  print(numbers);             // [2, 4, 5, 6]

  numbers.clear();
  print(numbers);             // []
  print(numbers.isEmpty);     // true
}

```

### Set

- 중복되지 않은 값들이 묶인 형태
- Set 필수 구성 요소 : Set<타입> + 변수 = {요소}
```dart
Set<int> numbers = {1, 2, 3, 4, 5};
```

  - 모든 요소의 타입은 같아야 한다.
- Dart 문법의 타입 추론을 활용하면 Set없이 만들 수 있다.
```dart
var numbers = {1, 2, 3, 4, 5};             // Set 요소의 변동이 있는 경우
const names = {'Alice', 'Bob', 'Henry'};   // Set 요소의 변동이 없는 경우
```

  - 요소에 변동이 있는 경우 var로 선언이 가능, 변동이 없는 경우 final/const로 선언이 가능
- Set는 List와 다르게 Index라는 개념이 없어서 요소들의 순서가 보장되지 않는다.
- set는 중복을 허용하지 않아 주로 중복된 값을 제거하고 싶을 때 많이 사용된다.
- 자주 사용되는 옵션은 List와 유사하지만, Index를 사용하는 옵션은 사용 불가이다.
- Set에서 자주 사용되는 옵션
### Map

- Key 와 Value 가 묶인 하나의 쌍으로 이루어진 형태이다.
- 필수 구성 요소 : Map<key 타입, value 타입> + 변수 = {key: value};
```dart
Map<String, String> people = {'Alice': 'Teacher', 'Bob': 'Student'};
```

  - 키와 값을 서로 다른 타입으로 구성할 수 있지만, 각각 타입은 같아야 한다.
  - 키는 중복될 수 없지만, 값은 중복될 수 있다.
- Dart 문법의 타입 추론을 활용하면 Map없이 만들 수 있다.
```dart
var people = {'Alice': 'Teacher', 'Bob': 'Student'};   // Map 요소의 변동이 있는 경우
const animals = {'Dog': 3, 'Cat': 5};                  // Map 요소의 변동이 없는 경우
```

  - 요소에 변동이 있는 경우 var로 선언이 가능, 변동이 없는 경우 final/const로 선언이 가능
- Map에서 자주 사용되는 옵션
```dart
void main() {
  Map<String, dynamic> dog = {
    'name': '초코',
    'age': 3,
  };

  print(dog['name']);              // 값 검색 → '초코'

  dog['age'] = 4;                  // 값 수정
  dog['isCat'] = false;            // 새 키-값 추가

  print(dog.length);               // 3       key-value 갯수
  print(dog.isEmpty);              // false

  dog.remove('isCat');             // 키 삭제

  print(dog.containsKey('name'));  // true    name이라는 키의 유무

  print(dog.keys);                 // (name, age)
  print(dog.values);               // (초코, 3)
}
```
