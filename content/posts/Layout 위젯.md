---
title: "Layout 위젯"
status: "published"
date: "2026-04-04"
type: "posts"
---

# Layout 위젯

# 📚 위젯의 종류 2

## Layout 구성 위젯

### Container 위젯

- Container 위젯은 Flutter에서 가장 일반적으로 사용되는 사각형 박스 위젯이다.
- 자주 사용되는 Container의 옵션
```dart
Container(
  // 내부 여백(padding)을 좌우 20픽셀씩 줌
  padding: const EdgeInsets.only(
    left: 20,
    right: 20,
  ),

  // 컨테이너의 꾸밈 설정을 위한 decoration (BoxDecoration 클래스 사용)
  decoration: BoxDecoration(
    // 배경을 그라데이션으로 설정 (LinearGradient 클래스)
    gradient: LinearGradient(
      colors: [
        // 색상: 반투명 빨강 (opacity 0.7)
        Color.fromARGB(255, 255, 59, 98).withOpacity(0.7),
        // 색상: 진한 빨강 (불투명)
        Color.fromARGB(255, 255, 59, 98)
      ],
      // 그라데이션 시작 위치 (왼쪽 위)
      begin: Alignment.topLeft,
      // 그라데이션 끝 위치 (오른쪽 아래)
      end: Alignment.bottomRight,
    ),

    // 모서리를 둥글게 만들기 (반지름 10)
    borderRadius: BorderRadius.circular(10),

    // 그림자 효과 (BoxShadow 리스트)
    boxShadow: [
      BoxShadow(
        // 그림자 색상 (반투명 빨강)
        color: Color.fromARGB(255, 255, 59, 98).withOpacity(0.5),
        // 그림자가 퍼지는 정도
        spreadRadius: 5,
        // 그림자의 흐림 정도 (부드럽게 만듦)
        blurRadius: 7,
        // 그림자 위치를 살짝 아래로 이동 (x=0, y=3)
        offset: Offset(0, 3),
      ),
    ],
  ),

  // 컨테이너의 너비를 200픽셀로 설정 (double 타입 변수)
  width: 200,

  // 컨테이너의 높이를 150픽셀로 설정 (double 타입 변수)
  height: 150,

  // 자식 위젯 (child)로 Center 위젯을 넣음
  child: Center(
      // Center 안에 텍스트 위젯
      child: Text(
    'Container',
    // 텍스트 스타일: 흰색 글씨
    style: TextStyle(color: Colors.white),
  )),
),
```

### SizedBox 위젯

- SizedBox 위젯은 고정된 크기를 가진 박스를 만드는 위젯이다. 
- child를 감싸 특정 크기를 강제하거나 빈 공간을 만드는데 사용할 수 있다.
- Container와 SizedBox 위젯의 차이점
  - Container는 크기, 색상, 그림자 등 다양한 기능을 가지지만, SizedBox는 단순 크기만 
지정하거나, 빈 공간을 만드는 게 주 목적
  - Container는 여러 속성을 계산하느라 무겁고 복잡할 수 있지만, SizedBox는 단순히 
크기만 조절하기 때문에 가볍고 빠르다.
### Row & Column 위젯

- Row 위젯은 children 위젯들을 가로로 나열해 주는 위젯이다.
- Column 위젯은 children 위젯들을 세로로 나열해 주는 위젯이다.
- Row 위젯과 Column 위젯의 필수 구성요소는 Row() / Column() + children[] 이다.
```dart
Row(                              //Row() 또는 Column()
  children: [
    Text('첫 번째'),
    Icon(Icons.star),
  ],
)
```

  - Row와 Column은 모두 부모 위젯의 크기 제한을 받을 수 있음
- Row & Column 위젯 에서 자주 사용되는 옵션
```dart
Row(
  mainAxisAlignment: MainAxisAlignment.위치,
)

// 위치에 사용될 수 있는 값
MainAxisAlignment.start             // 시작 지점으로 정렬
MainAxisAlignment.end               // 끝 지점으로 정렬
MainAxisAlignment.center            // 가운데 정렬
MainAxisAlignment.spaceBetween      // 위젯들 사이에만 동일한 간격
MainAxisAlignment.spaceAround       // 위젯 주변에 동일한 간격
MainAxisAlignment.spaceEvenly       // 모든 공간이 균등하가 나눠짐
```

  - 직접 간격을 조절하고 싶다면 SizedBox를 이용해야 함
  - spaceBetween, spaceAround, spaceEvenly는 자동으로 균등한 간격을 만들어 주기 
때문에직접 간격 크기를 조절 불가
### Expanded 위젯

- Expanded 위젯은 Row, Column, Flex 위젯에서만 사용 가능하고, 자식 위젯이 가능한 남은 공간을 꽉 채우도록 만들어 주는 위젯이다.
- Expanded 위젯의 필수 구성 요소는 Expanded() + child 이다.
  - Expanded는 자기 안에 단 하나의 위젯만 감싸는 역활을 하기 때문에 children을 쓸 수 없다.
- Expanded 위젯에서 자주 쓰는 옵션
  - flex : 비율을 통해 남은 공간을 채워 넣는 숫자값이다.
```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  children: [
    Expanded(
      flex: 1,
      child: Container(
        height: 40,
        color: Colors.yellow,
        margin: const EdgeInsets.all(5),
      ),
    ),
    Expanded(
      flex: 3,
      child: Container(
        height: 40,
        color: Colors.blue,
        margin: const EdgeInsets.all(5),
      ),
    ),
    Container(
      width: 40,
      height: 40,
      color: Colors.red,
      margin: const EdgeInsets.all(5)
    ),
  ],
),
// 왼쪽에 고정된 40 너비 빨간박스
// 빨간 박스를 제외하고 남은 공간 중 1/4 를 노랑 박스
// 빨간 박스를 제외하고 남은 공간 중 3/4 를 파란 박스
// spaceEvenly를 통해 세 위젯 사이에 동일한 간격
```

![](/images/05_Layout 위젯/img_01.png)

### Stack & Positioned 위젯

- Stack 위젯은 위젯을 겹쳐서 쌓을 수 있게 해주는 레이아웃 위젯이다.
- Stack 위젯의 필수 구성요소는 Stack() + children[] 이다.
```dart
Stack(
  children: [
    // 여기에 겹쳐서 쌓을 위젯들
  ],
)
```

- Stack 위젯에서 자주 사용되는 옵션
  - fit : 자식 위젯들을 얼마나 꽉 채울지, 그대로 둘지 결정하는 속성이며, 설정값은 총 3가지가 있다.
- Positioned 위젯은 Stack 안에서 위젯의 정확한 위치와 크기를 조절하기 위해서 사용한다.
  - 기본적으로 자식들은 좌측 상단(0,0)을 기준으로 쌓인다.
  - Positioned 위젯은 오직 Stack 위젯 안에서만 쓸 수 있다.
- Positioned에서 자주 사용되는 옵션
  - top, left, right, bottom은 동시에 2개만 지정해도 된다.
# 💡 문제 해결

## 학습하면서 생각이 든 의문점 및 해답(ChatGPT)

- Stack를 공부하면서 위젯이 왼쪽, 오른쪽, 상단, 하단에서 부터 움직는거에 대한 궁금증
**Q1. Stack 위젯을 정중앙에 위치하게 하고싶으면 Positioned를 사용하여 상하좌우를 조정해서 
      지정해야 하는 걸까?**

  A1. Center 위젯으로 Stack을 감싸면 자식 위젯들을 앱 화면 정중앙에 배치 할 수 있다. 
      즉, Center 위젯은 자식 위젯을 앱 화면의 정중앙에 배치하는 역할을 한다.

**Q2. 그럼 정중앙에서 조금 위나 아래, 좌, 우로 옮기고 싶으면 어떻게 할까?**

  A2. Center 대신 Align을 이용하여 비율로 위치를 조정하여 대략적인 위치 이동이 가능하며, 
      Transform.translate를 사용하여 Offset(x, y)로 정확한 px단위로 이동이 가능하다.

## 코드 작성 시 문제점 + 해결방안

- 학습 코드를 Dart Pad에서나 VScode에서 시뮬레이션 하기 위해선 필수 적인 조건이 필요하다.
- 다음 학습 코드를 그냥 실행시키면 에러가 뜨며 실행할 수 없다.
```dart
Row(
  mainAxisAlignment: MainAxisAlignment.center,
  children: List.generate(
    5,
    (index) => Container(
      width: 40,
      height: 40,
      color: Colors.red,
      margin: const EdgeInsets.all(5),
    ),
  ),
),
```

- 앱을 실행하려면 기본적으로 void main() ⇒ runApp()으로 실행을 시작하고, StatelessWidget 또는 
StatefulWidget으로 만든 설계도 클래스에서 MaterialApp → Scaffold → 화면 내용 구조를 따라야
에러 없이 시작할 수 있다.
  1. 앱 실행의 시작점
```dart
void main() {                  // main() 함수는 Dart 프로그램의 시작점
  runApp(MyApp());             // runApp() 함수는 Flutter 앱을 화면에 그려주는 함수
}                              // MyApp()은 실제 앱의 설계도 클래스
```

  1. 앱의 설계도 클래스 (StatelessWidget 또는 StatefulWidget)
```dart
class MyApp extends StatelessWidget {     // MyApp는 앱 전체를 정의하는 클래스
  @override
  Widget build(BuildContext context) {
    return MaterialApp(                   // MaterialApp을 리턴해야 스타일이 적용됨
```

  1. 첫 화면 구조 (home: Scaffold)
```dart
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('제목')),
        body: ...
      ),
    );
    
    // home: 앱이 시작될 때 첫 화면 위젯
    // Scaffold는 Flutter 앱의 화면 뼈대를 구성하는 기본 틀
```

# 📒 New 용어 정리
