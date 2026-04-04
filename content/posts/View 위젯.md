---
status: "published"
date: "2026-04-04"
type: "posts"
---

# View 위젯

# 📚 위젯의 종류 1

## View 위젯

### PageView 위젯

- PageView 위젯은 가로 또는 세로로 스와이프 할 수 있는 페이지를 만들기 위한 위젯이다.
  - PageView 위젯의 기본으로 넘기는 방향은 가로 스와이프임
  - 세로로 스와이프 하는 기능은 다음 코드를 사용함
```dart
scrollDirection: Axis.vertical,
```

- PageView 위젯의 구성 요소
  - PageView(위젯) + children[](위젯을 구성하는 페이지들) + 페이지를 넘기는 기능(선택 옵션)
  - 선택 옵션을 넣지 않을 경우 가로로 스와이프 
```dart
PageView(
  children: [
    위젯1, // 첫 번째 페이지
    위젯2, // 두 번째 페이지
    ...
  ],
)
```

- 자주 사용되는 PageView 위젯의 옵션
  - controller : 버튼을 만들어 원하는 페이지를 이동할 수 있게 한다.
  - pageSnapping : 페이지를 넘길 때 고정될지 말지 정한다.
  - onPageChanged : 페이지가 바뀔 때 알림이 실행되게 한다.
### ListView 위젯

- ListView 위젯은 스크롤 가능한 위젯 리스트를 보여주는 위젯이다.
- ListView 위젯의 구성 요소
  - ListView(위젯) + children[](위젯을 구성하는 리스트들) + 스크롤 기능(선택 옵션)
  - 선택 옵션을 넣지 않을 경우 세로로 스크롤
```dart
ListView(
  children: [
    위젯1, // 첫 번째 페이지
    위젯2, // 두 번째 페이지
    ...
  ],
)
```

- 자주 사용되는 ListView 위젯의 옵션
```dart
ListView(
  scrollDirection: Axis.vertical,   // ① 스크롤 방향 전환
  reverse: false,                   // ② 리스트 뒤집기
  padding: EdgeInsets.all(8),       // ③ 안쪽 여백
  physics: BouncingScrollPhysics(), // ④ 스크롤 감도, 느낌 변화
  cacheExtent: 500,                 // 미리 만들어 둘 영역(픽셀)
  controller: ScrollController(),   // 스크롤 상태를 직접 제어
```

### GridView 위젯

- GridView 위젯은 격자 모양으로 위젯 리스트를 나열하고 스크롤이 가능한 위젯이다.
- GridView 위젯의 구성 요소
  - GridView(위젯) + gridDelegate: + 클래스 인스턴스 + 클래스 인스턴스 매개변수 + children[](위젯을 구성하는 리스트들) + 스크롤 기능(선택 옵션)
  - 선택 옵션을 넣지 않을 경우 세로로 스크롤
  - gridDelegate은 GridView의 가로 몇 칸, 칸 간격 등 규칙 지정한다.
```dart
import 'package:flutter/material.dart';

GridView(
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(     // 클래스 인스턴스
    crossAxisCount: 2, // 클래스 인스턴스 매개변수
  ),
  children: [
    Widget1(),
    Widget2(),
    Widget3(),
  ],
)
```

- 자주사용되는 GridView 위젯 배치 옵션
  - 주요 클래스 인스턴스와 매개변수
```dart
// 주요 필수 클래스 및 매개변수
SliverGridDelegateWithFixedCrossAxisCount()  // 가로 칸수를 고정해 격자 만듬
  crossAxisCount:                            // 가로 칸수 고정 매개변수

SliverGridDelegateWithMaxCrossAxisExtent()   // 각 칸 너비를 고정해 격자 만듬
  maxCrossAxisCount:                         // 각 칸 너비 고정 매개변수

// 그 외 매개변수
  mainAxisSpacing: 10,                       // 세로 간격 고정 매개변수
  crossAxisSpacing: 10,                      // 가로 간격 고정 매개변수
  childAspectRatio: 1.0,                     // 정사각형 칸 비율 고정 매개변수
```

### TabBar 위젯

- TabBar의 구성은 TabController클래스, TabBar위젯, TabBarView위젯의 위젯트리로 구성된다.
- TabBar의 필수 기본 구성 위젯 트리
  - 여기서 length 수, tabs의 리스트 수, children의 리스트 수는 같아야 한다.
```dart
TabController 또는 DefaultTabController
│   ├── length: 3               // 탭의 총 개수를 정하는 변수 (필수)
│   └── vsync: this,            // 애니메이션을 부드럽게 실행하기 위한 장치 (필수)
│
├── TabBar (위젯)
│   └── tabs: List<Tab>         // 각 탭 버튼들 (Tab은 위젯/클래스)
│       └── Tab(text: '이름')   // 탭 하나하나
│   └── 기타 옵션들 (선택)
│       ├── isScrollable: bool
│       ├── indicatorColor: Color
│       └── labelStyle: TextStyle
│
├── TabBarView (위젯)
    └── children: List<Widget>   // 각 탭에 해당하는 실제 화면들 (ex: HomePage() 등)
```

- TabController에서 자주 쓰는 변수 옵션
- TabBar 위젯에서 자주 사용되는 옵션
```dart
TabBar(
  controller: _tabController,                             // TabController 연결(필수)
  labelColor: Colors.blue,                                // 선택된 탭 글자 색
  unselectedLabelColor: Colors.grey,                      // 선택 안 된 탭 글자 색
  labelPadding: const EdgeInsets.symmetric(vertical: 20), // 선택된 탭 글자 패딩
  tabs: const [                                           // 탭 메뉴 리스트
    Text('메뉴1'),
    Text('메뉴2'),
    Text('메뉴3'),
  ],
)
```

# 💡 문제 해결

## 학습하면서 생각이 든 의문점 및 해답(ChatGPT)

- GridView 위젯의 기본 구성요소에서 클래스 인스턴스 매개변수에 대해 궁금점 발생
**Q1. SliverGridDelegateWithFixedCrossAxisCount() 매개변수가 crossAxisCount로 쓰던데**

        SliverGridDelegateWithMaxCrossAxisExtent() 매개변수가 crossAxisExtent가 아닌 이유?

  A1. 클래스가 어떤 역할을 하는지 이름으로 나타내고 그 역할에서 변하는 핵심값을 매개변수의 
      이름으로 정한다. 즉, SliverGridDelegateWithMaxCrossAxisExtent()에서 최대 너비까지가 
      변수로 발생하기 때문에 매개변수는 maxCrossAxisExtent로 와야한다.

- TabBar 위젯에 TabController에서 자주 쓰는 변수 옵션에서 Duraion 타입에 관하여
**Q2. animationDuration은 변수고 Duration이 타입이라고 했는데 클래스인 Duration가 타입 
      될 수 있을까?**

  A2. animationDuration라는 변수에 Duration 클래스의 객체를 넣어줘야 한다는 의미이다.
      즉, Duration의 객체가 되는 Duration(seconds: 1) 등의 값을 넣어줘야한다.

**Q3. TickerProvider 타입이 왜 필요할까? 아래 코드처럼 변수 선언하고 쓰면 되지 않을까?**

```dart
TickerProvider tool = _tool;
someFunction(tool: this);
```

  A3. vsync변수의 타입을 TickerProvider로 지정하는 이유는 변수 선언 시 타입과 이름을 
      한꺼번에 정하는 것도 있고 이렇게 하면 컴파일러가 그 변수에 어떤 종류의 객체가 와야 하는지       알 수 있다. 즉, vsync를 통해 값을 한번에 할당 할 수 있다.

```dart
vsync: this
```

# 🤔 회고

## 추가적으로 알게된 학습 내용

- 클래스는 타입으로 사용이 가능하다. (TIL : 2025-05-29 Dart 문법 기초 2 → 클래스에 보완)
```dart
Dog myDog; // Dog는 타입
myDog = Dog('초코', 2); // Dog 클래스의 객체(인스턴스)를 생성해서 대입
```

  - 예시
```dart
//클래스 = Dog
class Dog {
  // 멤버변수 (특성)
  String name = '초코';
  int age = 2;

  // 생성자
  Dog(this.name, this.age);

  // 메서드 (기능)
  void bark() {
    print('$name: 멍멍!');
  }
}

Dog dog1 = Dog('코코', 3);
Dog dog2 = Dog('초코', 2);

void main() {
  dog2.bark(); 
}

//출력 초코: 멍멍!
```

# 📒 New 용어 정리
