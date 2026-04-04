---
status: "published"
title: "Flutter StatefulWidget vs StatelessWidget 완벽 정리"
date: "2026-04-04"
tags: ["Flutter", "Widget", "기초"]
---

# Flutter StatefulWidget vs StatelessWidget

Flutter 개발을 시작하면 가장 먼저 마주치는 개념이 바로 StatefulWidget과 StatelessWidget이다.
둘의 차이를 명확히 이해하면 불필요한 리렌더링을 줄이고 성능 좋은 앱을 만들 수 있다.

## StatelessWidget

상태(State)가 없는 위젯이다. 한번 그려지면 외부에서 새로운 데이터를 받지 않는 한 다시 그려지지 않는다.

```dart
class MyText extends StatelessWidget {
  final String text;

  const MyText({required this.text});

  @override
  Widget build(BuildContext context) {
    return Text(text);
  }
}
```

**언제 쓰나?**
- 단순히 데이터를 보여주기만 할 때
- 버튼 클릭, 입력 등 내부 변화가 없을 때

## StatefulWidget

내부에 상태를 가지며, 상태가 바뀌면 `setState()`를 호출해 화면을 다시 그린다.

```dart
class Counter extends StatefulWidget {
  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('$count'),
        ElevatedButton(
          onPressed: () => setState(() => count++),
          child: Text('증가'),
        ),
      ],
    );
  }
}
```

**언제 쓰나?**
- 버튼 클릭, 입력값 변경 등 내부 상태가 바뀔 때
- 애니메이션, 타이머 등 시간에 따라 변하는 UI

## 핵심 비교

| | StatelessWidget | StatefulWidget |
|---|---|---|
| 상태 | 없음 | 있음 |
| 리렌더링 | 외부 변경 시만 | setState() 호출 시 |
| 성능 | 더 가벼움 | 상대적으로 무거움 |
| 사용 예 | 텍스트, 아이콘 | 카운터, 폼, 애니메이션 |

## 정리

가능하면 StatelessWidget을 쓰고, 내부 상태가 필요할 때만 StatefulWidget을 쓰는 것이 좋다.
무조건 StatefulWidget을 쓰는 습관은 성능 저하로 이어질 수 있다.
