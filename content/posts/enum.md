---
title: "enum"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 Enumerations(열거형)

## enum(Enumerations : 열거형)

- 사용자가 새로운 변수에 대한 타입을 만드는 키워드로 사용된다.
- 새로운 타입과 값을 정의하기 때문에 void함수 안에 사용할 수 없다.
  - enum는 타입 이름을 설정하고 {}안에 설정한 타입이 사용 가능한 값을 넣음
```dart
enum Weather {      // Weather은 타입 이름
  sunny,            // Weather 타입에 쓸 수 있는 값들 (sunny, cloudy,rainy)
  cloudy,
  rainy
}
```

- enum의 필수 구성 요소 : enum + 타입 이름 + {값1, 값2, …}
```dart
enum 타입 이름 {     // 임의의 타입 이름, 대문자로 시작
  값1,               // 이 타입으로 사용할 수 있는 값들
  값2,
  값3,
}
```

  - 값에 넣을 수 있는 요소들
  - 예식 코드
```dart
enum Weather {
  sunny('☀️'),                // 생성자의 선언 = ()로 인자가 필요
  cloudy('☁️'),               // 생성자가 없으면, ()인자는 불필요
  rainy('🌧️');

  // 1. 변수 선언
  final String icon;           // String 타입의 변수 icon 선언

  // 2. 생성자 선언
  const Weather(this.icon);    // Weather 타입의 값에서 인자를 this.icon으로 선언

  // 3. 메서드도 가능
  void describe() {
    print('오늘의 날씨는 $name, 아이콘은 $icon');
  }
}

void main() {
  Weather today = Weather.cloudy;

  print(today.icon);  // 출력: ☁️
  today.describe();   // 출력: 오늘의 날씨는 cloudy, 아이콘은 ☁️
}

```

### enum 활용

- enum외부 활용법
  - 예시 코드
```dart
enum Weather {
  sunny,
  cloudy,
  rainy
}

void main() {
  Weather today = Weather.cloudy;

  // 1. switch 문으로 분기
  switch (today) {
    case Weather.sunny:
      print("☀️ 해가 쨍쨍");
      break;
    case Weather.cloudy:
      print("☁️ 흐림");
      break;
    case Weather.rainy:
      print("🌧️ 비가 와요");
      break;
  }                      // 출력 : ☁️ 흐림

  // 2. if 문으로 조건 처리
  if (today == Weather.cloudy) {
    print("오늘은 흐려요");
  }                      // 출력 : 오늘은 흐려요

  // 3. .name 속성
  print(today.name);     // 출력: cloudy

  // 4. .index 속성
  print(today.index);    // 출력: 1

  // 5. values 속성 (모든 항목)
  print(Weather.values); // 출력: [Weather.sunny, Weather.cloudy, Weather.rainy]

  // 6. for문으로 반복 순회
  for (var w in Weather.values) {
    print('${w.index}: ${w.name}');
  }
}                        // 출력 : 0: sunny   1: cloudy   2: rainy

```

### enum과 List, Set의 비교

- enum은 고정된 항목들을 타입으로 정의하고, List와 Set은 값들을 모아 저장하는 컬렉션이다.
  - enum : 의미 있는 고정된 항목들을 타입으로 정의할 때
  - List : 순서 있는 데이터 모음을 다룰 때
  - Set : 순서 없고 중복 없는 데이터 모음을 다룰 때