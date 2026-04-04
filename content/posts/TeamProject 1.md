---
title: "TeamProject 1"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 쇼핑몰 앱

## 새롭게 알게 된 정보

**Column/Row + Align : Column 또는 Row에서 Text의 왼쪽과 오른쪽 정렬 같이 하는 방법**

- Align 위젯 사용하기
```dart
Column(
  crossAxisAlignment: CrossAxisAlignment.stretch, // 이거 중요!
  children: [
    Align(
      alignment: Alignment.centerLeft,
      child: Text('왼쪽 정렬'),
    ),
    Align(
      alignment: Alignment.centerRight,
      child: Text('오른쪽 정렬'),
    ),
  ],
)

```

  - CrossAxisAlignment.stretch : 자식들의 가로(세로) 길이를 최대로 늘려줌
- Alignment.위치 사용 방법
```dart
// 위치에 해당하는 값들
     (-1, -1)              (0, -1)              (1, -1)
     topLeft             topCenter            topRight

     (-1,  0)              (0,  0)              (1,  0)
   centerLeft             center             centerRight

     (-1,  1)              (0,  1)              (1,  1)
   bottomLeft           bottomCenter         bottomRight
```

**ListView.builder : 스크롤이 필요하고 항목이 많고 개수가 동적일 때 사용**

- Flutter에서 화면 스크롤의 대표적인 방식
- ListView.builder의 속성
  - itemCount: : 총 몇 개의 항목을 만들지 결정
  - itemBuilder: (context, index) {return 위젯; } : 각 항목을 어떻게 그릴지 결정index를 활용하여 위젯으로 만듬
```dart
ListView.builder(
  itemCount: items.length,      // 항목 개수는 = items라는 리스트의 길이
  itemBuilder: (context, index) {
    final item = items[index];  // index로 하나 꺼냄
    return ListTile(title: Text(item));
  },
)
```

**Dismissible : 자식 위젯을 좌우로 스와이프해서 제거할 수 있게 해주는 기능성 위젯**

- Dismissible 속성
- 예시 코드 
```dart
Dismissible(
  key: ValueKey('item1'),
  direction: DismissDirection.endToStart, // 오른쪽에서 왼쪽으로 스와이프 가능
  onDismissed: (direction) {
    // 스와이프 후 아이템 삭제 처리 등 실행
  },
  background: Container(
    color: Colors.red,
    alignment: Alignment.centerRight,
    padding: EdgeInsets.only(right: 20),
    child: Icon(Icons.delete, color: Colors.white),
  ),
  child: ListTile(
    title: Text('Item 1'),
  ),
);
```

**forEach : 컬렉션 요소를 하나씩 순회할 때 사용**

- List, set, Map등 컬렉션 안의 모든 요소에 대해 순회하여 내보냄
```dart
// List 또는 Set에서 다음과 같이 사용
Set<int> numbers = {1, 2, 3};

numbers.forEach((num) {
  print(num);
}); 
// 출력 
// 1
// 2
// 3

// Map에서 다음과 같이 사용
Map<String, int> scores = {'math': 90, 'english': 80};

scores.forEach((key, value) {
  print('$key: $value');
});
// 출력
// math: 90
// english: 80
```

- forEach와 for-in의 차이점
**entries : Map안의 각각의 항목(key-value)을 객체처럼 따로 다루고 싶을 때 사용**

- Map에 있는 하나의 항목을 따로 다룰 때 사용되며 이때 따로 다루는 Map의 타입은 MapEntry타입
```dart
Map<String, int> fruits = {
  'apple': 3,
  'banana': 5,
};
for (MapEntry<String, int> entry in fruits.entries) {};
// 이때 entry의 타입은 MapEntry<String, int>이다.
```

  - entries와 반대로 fromEntries()는 Map으로 합쳐주는 메서드
- 자주 사용되는 곳
  - for-in으로 루프에서 Map을 순회할 때
```dart
Map<String, int> fruits = {
  'apple': 3,
  'banana': 5,
};
// for-in 순회 시 사용, fruits에 있는 항목을 하나씩 꺼내서 entry에 넣음
for (MapEntry<String, int> entry in fruits.entries) {
  print(entry.key);    // 'apple', 'banana'
  print(entry.value);  // 3, 5
}
```

    - 반복문이 끝나면 entry에는 뭐가 남을까?
```dart
// 하나씩 순회하면 entry에 저장되기 때문에 마지막으로 순회한 key-value가 저장
print(entry);
// 출력 : banana: 5
```

  - Map데이터를 List로 바꿀 때
```dart
Map<String, int> map = {'apple': 3, 'banana': 5};

// MapEntry 타입의 리스트로 만들기
List<MapEntry<String, int>> entryList = map.entries.toList();

print(entryList);  // [MapEntry(apple: 3), MapEntry(banana: 5)]
```

  - 조건에 맞는 새로운 Map을 만들 때
```dart
Map<String, int> map = {'apple': 3, 'banana': 5, 'cherry': 7};

// 값이 4 이상인 항목만 추출해서 새로운 Map 생성
Map<String, int> filteredMap = Map.fromEntries(
  map.entries.where((entry) => entry.value >= 4),
);

print(filteredMap);  // {banana: 5, cherry: 7}
```

  - 빈 맵에 하나씩 추가할 때
```dart
Map<String, int> newMap = {};

for (var entry in map.entries) {
  if (entry.value >= 4) {
    newMap[entry.key] = entry.value;
  }
}

print(newMap);  // {banana: 5, cherry: 7}
```

## 쇼핑몰 서비스 (장바구니 페이지)

![](/images/26_TeamProject 1/img_01.png)

![](/images/26_TeamProject 1/img_02.png)

![](/images/26_TeamProject 1/img_03.png)

### 나의 과제 시나리오

- 목표 : 쇼핑몰 앱 장바구니 페이지 구현하기
**기본 조건**

- 장바구니에 담기 한 상품들 장바구니 페이지로 불러오기
- 수량 변경 시 수량과 금액도 함께 변경되도록 구현
- 장바구니에 담긴 상품이 없을 경우 장바구니가 비었다는 표시 보이고 구매하기 버튼 비활성화
- 상품을 삭제할 경우 오른쪽에서 왼쪽으로 스와이프를 통해 삭제
**UI 디자인 명세 :**

**Color :**

- Black Text Color : 1C1B1F
- Deactivate Button : A3A3A3
- White Text Color : F6F6F6
- White Background Color : FFFFFF
- Buy Button Color : 233758
- Count Button Color : 5A7194
- Count Background Color : E8E6DB
- AppBar Color : 5A7194
- Empty Cart Icon Color : 5A7194
- Empty Text Color : A3A3A3
**Appbar :**

- 크기 ⇒ 기본
- 모양 ⇒ 아래쪽 모서리 둥글기 8
- 글씨 크기 ⇒ 기본
- 글씨 굵기 ⇒ 진하게
**body**

- body 전체 패딩 ⇒ 양옆 : 15
**Item List Box :**

- 전체 패딩 ⇒ 위아래 : 10
- 상품 전체 박스 ⇒ 높이 : 180, 너비 꽉차게, 모든 모서리 둥글기 : 15
-  상품 정보 부분 ⇒ 패딩 전체 : 15
- 상품 이미지 위젯 ⇒ 모든 모서리 둥글기 : 10, 너비 : 100, 높이 : 140
- 이미지와 설명 사이 공간 ⇒ 20
- 상품 이름 글씨 ⇒ 크기 :  24, 굵기 진하게
- +/- 수량 버튼 ⇒ 너비 : 30, 높이 : 30, 모든 모서리 둥글기 : 12, 아이콘 : 꽉차게
- 수량 박스 와 글씨 ⇒ 너비 : 50, 높이 : 30, 글씨 크기 : 15, 굵기 : 진하게
**Bottom Box :**

- 아래 박스 ⇒ 높이 : 140, 너비 : 꽉차게, 그림자 : (위로 2, 밝기 : 2, 범위 : 2), 모서리 둥글기 위로 : 10
- 글씨 ⇒ 크기 : 20, 굵기 : 진하게
- 구매하기 버튼 ⇒ 너비 : 250, 높이 : 45, 글씨 굵기 : 진하게
## 구성하기

### 컴포넌트화

- 레이아웃
  - MyCartPage ⇒ 전체적으로 세로로 배치(Column 사용)
- 나누어진 레이아웃 별 필요한 요소 구상하기
**MyCartPage**

  - 최상단 ⇒ AppBar
  - 상품 리스트 박스 ⇒ CartItemList
  - 상품 리스트 박스에 수량 박스 ⇒ CountButton
  - 하단 박스 ⇒ BottomButton
  - 빈 장바구니 페이지 ⇒ EmptyItem
- 요소 별 필요한 위젯 구상하기
**MyCartPage**

  - AppBar ⇒ Text
  - CartItemList ⇒ Container(Row(Image, Column(Text, CountButton))
  - CountButton ⇒ Column(ElevatedButton, Container(Text), ElevatedButton, Text)
  - BottomButton ⇒ Container(Column(Row(Text, Text), ElevatedButton))
  - EmptyItem ⇒ Colum(Icon, Text)
- 디렉토리 분리
```dart
📁 my_cart_page/
├── my_cart_page.dart
└── 📁 widgets/
     ├── bottom_button.dart
     ├── cart_item_list.dart
     ├── count_button.dart
     └── empty_item.dart
```

### 상태 관리 구성하기

- My_Cart_Page
**리스트 수량 변경 또는 삭제 시 다시 빌드**

  - ConutButton → MyCartPage (총 가격, 상품 총 가격, 수량)
## Team Project 구현 완료

### Team Project Pocket Poster

- 팀 대시보드 : Untitled 
- 최종 APP 구현 완료 
## 트러블슈팅 과정

**dismissible에서 Map 타입 사용**

- 문제 : 
  - dismissible에서 index를 활용하여 사용하는데 Map타입 itemToCart에  index를 사용하니 에러가 발생
- 원인 : Map타입은 index가 없어 사용이 불가능
- 해결 : 
  1. forEach방법으로 Map에 있는 Key와 value를 한 쌍으로 꺼내서 LIst<Map<Poster, int>>타입으로 리스트에 저장하는 방법 ← 이방법 선택
  1. dismissible에서 Map타입을 사용할 때 .toList로 리스트화 시켜서 dismissible이 실행 될 때만 리스트로 만들어서 사용
**상품의 수량을 변경 중 발생 된 트러블**

- 문제 : 수량을 변경하기 위해서 Map의 value를 꺼내서 변경할 필요가 발생
- 원인 : 
  - foEach를 활용하여 Map타입을 리스트에 저장하는 방식을 사용했는데 value를 꺼내기 위해서 리스트에서 key가 같은  항목을 찾고 그 key의 value를 꺼내는 작업이 필요함
- 해결 : 
  - dismissible에서 Map 타입을 사용할 때 .toList로 리스트화 시켜서 dismissible이 실행 될 때만 리스트로 만들어서 사용하고 나머지 상품 관리는 Map타입으로 진행
**상품의 수량을 변경 중 발생 된 트러블 2**

- 문제 : CartListEntries가 Dismissible위젯 안에서 인식되지 않아 ‘Underfined name’ 에러 발생
- 원인 : 
  - final cartListEntries = cartList.entries.toList();를 _MyCartPageState클래스 안에서 선언하여 Dismissible 위젯을 사용하는 함수를 불러왔을 때 제대로 인식되지 않음
- 해결 : 
  - final cartListEntries = cartList.entries.toList();를 Dismissible 위젯을 사용하는 함수 notEmptyCart에서 선언하여 위젯 안에서 사용이 가능하도록 변경
**CartItemList에 상품 정보를 받자 오류 발생 **

- 문제 : CartItemList(cartPoster: cartListEntries[index])으로 넘겨 줬더니 오류 발생
- 원인 : 
  - cart.entries.toList를 하게되면 타입이 Map<Poster, int>에서 MapEntry<Poster, int>타입으로 변경되어 받아야 하는 타입인 Map<Poster, int>와 타입이 맞지 않아 오류가 발생
- 해결 : 받아야 하는 타입을 Map<Poster, int>에서 MapEntry<Poster, int>타입으로 변경