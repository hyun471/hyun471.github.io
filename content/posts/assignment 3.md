---
title: "assignment 3"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 기차 예매 서비스 구현하기

## 새롭게 알게 된 정보

**조건 ? 참일 때 값 : 거짓일 때 값; : 삼항 연산자**

- 조건문을 짧게 쓰는 축약형 if문
```dart
int num = 4;
String result = (num % 2 == 0) ? "짝수" : "홀수";
print(result); // 짝수
```

- 삼항 연산자는 한 줄로 끝나서 간결하지만, 복잡한 조건이나 여러 분기가 있으면 if문을 사용
**decoration: BoxDecoration(border: Border(테두리 옵션)) : 테두리에 색, 두께 넣기**

- 테두리 방향 : Border(top:, left:, right:, bottom: 중 선택해서 사용 BorderSide())
- 테두리 색 : BorderSide(color: Colors.색)
- 테두리 두께 : BorderSide(width: 두께(기본 0.0 픽셀))
** ...[] : ...는 리스트를 풀어서 밖에 리스트에 넣는 연산자**

```dart
children: [
  for (int i = 1; i <= 5; i++) ...[
    SeatList('$i'),
    if (i != 5) SizedBox(height: 8),
  ]
] // 결과 children [SeatList(1), SeatList(2), SeatList(3), ... SeatList(5)]
```

**Navigator.push(), Navigator.pop() : 스택 쌓기 및 최근 스택 제거**

- Navigator.push() : 현재 페이지 위로 새로운 스택(페이지)을 쌓고 데이터를 넘겨줌
```dart
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => NextPage()),
);
```

- await Navigator.push() + Navigator.pop : 결과 값만 넘겨 받을 수 있음 
```dart
final result = await Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => SelectPage()),
);
// 다른 페이지
Navigator.pop(context, '선택한 값');
```

- Navigator.pushName() : MaterialApp에서 routes에 스택을 등록하고 이름을 호출하여 사용  ⇒ 자주 사용
  - 호출 시 호출한 페이지를 기존 스택 위에 쌓음
```dart
// MaterialApp에 routes에 스택 등록
routes: {
  '/': (context) => HomePage(),
  '/settings': (context) => SettingsPage(),
}
// 호출
Navigator.pushNamed(context, '/settings');
```

- Navigator.pushNameAndRemoveUntil() : 기존 pushName 기능에 기존 스택을 전부 지우는 기능을 추가한 버전
  - 호출 시 호출한 페이지로 이동하며 기존 스택을 전부 제거 
```dart
Navigator.pushNamedAndRemoveUntil(
  context,
  '/homePage',           // 이동할 경로
  (Route<dynamic> route) => false,  
);  // false면 모든 이전 경로 제거, ture면 모든 경로 유지
```

- Navigator.pop() : 값을 전달하며 가장 최근 스택 제거, 연속으로 사용 가능  ⇒ 자주 사용
```dart
Navigator.pop(context, '결과값');  // 한 단계 뒤로
Navigator.pop(context, '결과값');  // 두 단게 뒤로
```

- Navigator.popUntil() : 조건을 만족할 때까지 pop  ⇒ 자주 사용
```dart
Navigator.popUntil(context, ModalRoute.withName('/home'));
// /home이 나올 때까지 계속 pop
```

- 라우터 사용 시 데이터는 전달 및 받기
  - 데이터 보내기 : arguments를 통해 전달
  - 데이터 받기 : ModalRoute로 꺼내기
```dart
Navigator.pushNamed(
  context,
  '/seat',
  arguments: {'seat': 'A3', 'price': 12000}
);
// 데이터 전달 (String, Map, List 등 어떤 타입도 가능)

 @override
  Widget build(BuildContext context) {
    final args = ModalRoute.of(context)!.settings.arguments as String;
    final seat = args['seat'];
    final price = args['price'];
    // 데이터 받기 (arguments는 Object? 타입이라 as를 써 형변환 해줘야 안전))
```

**테마 적용**

- MaterialApp에서 theme 모드 설정과 theme를 설정함
```dart
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        themeMode: ThemeMode.system,  // 테마 모드 설정 (시스템, 라이트, 다크 등)
        theme: lightTheme,            // 기본 테마에 사용될 테마
        darkTheme: darkTheme,         // darkTheme에 사용될 테마
        home: HomePage());
```

- 새로운 테마 파일에 테마에 적용될 내용을 구성
  - 테마의 주요 컨셉 색만 적용해도 알아서 색 조합 맞게 적용이 됨
  - 원하는 색을 입히고 싶을 땐, 수동으로 적용해야 함
  - 기본 배경 색과 글자의 경우 알아서 적용
```dart
final lightTheme = ThemeData(              // lightTheme에 사용될 내용
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.purple,            // 테마의 주요 컨셉 색
      brightness: Brightness.light,        // 테마의 주요 컨셉 밝기
    ),
    
    // 명칭으로 구분하는 색 (다른 용도로도 사용가능)
    hoverColor: Colors.white,              // 마우스 올렸을 때 배경색
    dividerColor: Colors.grey[200],        // 구분선 색상
    highlightColor: Colors.purple,         // 효과 시 나타날 색(예: 터치 시 색)
    focusColor: Colors.grey[200],          // 포커스 시 색
    
    // 버튼, 실린더 등 위젯 자체에 터치색, 테두리 색, 등 색 적용
    elevatedButtonTheme: ElevatedButtonThemeData(
        style: ButtonStyle(
      backgroundColor: WidgetStatePropertyAll(Colors.purple),
      foregroundColor: WidgetStatePropertyAll(Colors.white),
    )));
    // 위젯 자체에 색을 적용하면 자동으로 색이 변경됨
```

- 테마 색을 적용해야 할 부분에 색 대신 테마 색 입력
```dart
  @override
  Widget build(BuildContext context) {
    return Container(
      height: 120,
      width: double.infinity,
      decoration: BoxDecoration(
          color: Theme.of(context).hoverColor,  // 색을 넣을 부분에 테마색 넣기
 
 // 이러면 이 Container는 해당 테마의 hoverColor의 색으로 나옴
```

## 기차 예매 서비스

![](/images/25_assignment 3/img_01.png)

![](/images/25_assignment 3/img_02.png)

![](/images/25_assignment 3/img_03.png)

![](/images/25_assignment 3/img_04.png)

### 과제 시나리오

- 목표 : 기차 예매 시스템 UI와 아래 기능 구현하기
**기본 조건**

  - 프로젝트 명은 flutter_train_app
  - 출발역, 도착역을 선택할 수 있는 초기 화면의 이름은 HomePage
  - 기차역 리스트 보여주고 선택할 수 있는 화면의 이름은 StationListPage
  - 좌석 선택 화면은 SeatPage
  - 선택 가능 기차역은 수서, 동탄, 평택지제, 천안아산, 오송, 대전, 김천구미, 동대구, 경주, 울산, 부산 으로 총 11개 역
**UI 디자인 명세 :**

**HomePage**

1. [앱바] : 기차 예매 글자에 별도의 스타일 지정 X
1. [body 영역] :
  - background 색상 ⇒ Colors.grey[200]
  - body내의 위젯들은 세로 가운데 정렬
1. [출발역, 도착역 감싸고 있는 박스] :
  - 높이 ⇒ 200, 색상 ⇒ Colors.white, 모서리 둥글기 ⇒ 20
1. [출발역 글자] :
  - 크기 ⇒ 16, 색상 ⇒ Colors.grey, 두께 ⇒ FontWeight.bold
1. [선택 글자] : 크기 ⇒ 40
1. [세로선] : 
  - 너비 ⇒ 2, 높이⇒ 50, 색상 ⇒ Colors.grey[400]
1. [좌석 선택 버튼] :
  - 모서리 둥글기 ⇒ 20, 버튼 색상 ⇒ Colors.purple, 
  - 글자 색상 ⇒ Colors.white, 글자 크기 ⇒ 18, 글자 두께 ⇒ FontWeight.bold
1. [비고] :
  - 좌석 선택 버튼과 출발역 및 도착역 감싸고 있는 박스의 간격 ⇒ 20
  - 도착역 부분의 글자 스타일은 출발역 부분과 동일
  - Scaffold body의 padding은 ⇒ 20
**StationListPage**

1. [앱바] : 글자에 별도의 스타일 지정 X
1. [뒤로가기 버튼] : 별도로 선언하지 않고 APPBar에서 기본으로 나오는 버튼 사용
1. [기차역 이름] : 크기 ⇒ 18, 두께 ⇒ FontWeight.bold
1. [기차역 이름 감싸는 영역] : 높이 ⇒ 50, 테두리 ⇒ 아래에만 Colors.grey[300]!
**SeatPage**

1. [앱바] : 글자에 별도의 스타일 지정 X
1. [출발역, 도착역 글자] :
  - 두께 ⇒ FontWeigt.bold, 색상 ⇒ Colors.purple, 크기 ⇒ 30
1. [화살표 아이콘] : 
  - 아이콘 데이터 ⇒ Icons.arrow_circle_right_outlined, 크기 ⇒ 30
1. [좌석 상태 텍스트] : 별도의 스타일 X
  - 좌석 상태 박스와 텍스트 간격 ⇒ 4, 선택됨 / 선택 안됨 간격 ⇒ 20
1. [좌석 상태 박스] :
  - 너비 ⇒ 24, 높이 ⇒ 24, 모서리 둥글기 ⇒ 8
  - 색상-선택됨 ⇒ Colors.purple, 선택 안됨 ⇒ Colors.grey[300]!
1. [좌석 행/열 레이블] : 글자 크기 ⇒ 18, 글자를 감싸는 박스 크기 ⇒ 50 * 50
1. [좌석 위젯] :
  - 너비 ⇒ 50, 높이 ⇒ 50, 모서리 둥글기 ⇒ 8
  - 색상-선택됨 ⇒ Colors.purple, 선택 안됨 ⇒ Colors.grey[300]!
  - 좌석 가로 간격 ⇒ 4, 좌석 세로 간격 ⇒ 8, 총 좌석 행 갯수 ⇒ 20
1. [예매하기 버튼] : 
  - 모서리 둥글기 ⇒ 20 , 버튼 색상 ⇒ Colors.purple
  - 글자 색상 ⇒ Colors.white, 글자 크기 ⇒ 18, 글자 두께 ⇒ FontWeight.bold
1. [리스트뷰 영역] :
  - 위/아래로 패딩 ⇒ 20
  - 내용 가운데 정렬
  - 각 행마다 가운데 행 번호 출력
**필수 기능 :**

**HomePage**

1. [출발역/도착역 선택] :
  - 터치 시 StationListPage로 이동, StationListPage에서 역 선택 시 선택한 역으로 변경
  - 선택한 역이 없는 초기 상태에는 ‘선택’ 문구 출력
1. [좌석 선택 버튼] :
  - 터치 시 SeatPage로 이동, 역이 선택되지 않을 경우 이동하지 않음
**StationListPage**

1. [앱바 타이틀] : 
  1. HomePage에서 출발역 클릭 시 출발역 출력, 도착역 클릭 시 도착역 출력
1. [뒤로가기 버튼] : 아무런 값을 돌려주지 않고 뒤로가기
1. [기차역 리스트] : 고정된 11개의 역을 출력, 해당 역 터치 시 역 이름 반환하고 HomePage로 이동
**SeatPage**

1. [출발역/도착역] : HomePage에서 선택한 역을 전달 받아 출력
1. [좌석 상태] : 초기 상태는 모두 회색, 좌석 터치 시 보라색으로 변경
1. [예매하기] : 
  - 선택된 좌석이 없으면 반응X
  - 선택된 좌석이 있으면 showCupertionoDialog 출력
  - 취소 시 Dialog 제거 
  - 확인 시 HomePage로 이동
**도전 기능 :**

**위젯 컴포넌트화**

- 앱 내 반복적으로 사용되는 부분 또는 너무 긴 부분을 별도의 위젯으로 분리하여 재사용
**UX를 고려한 기능**

- 출발역에서 선택한 역은 도착역 선택 리스트에서 제외, 반대 경우도 동일
**다크 테마 적용**

- 앱에 타크테마 적용
**자신만의 기능 추가**

**출발역과 도착역 위치 바꾸기**

- 스위치 아이콘을 누르면 출발역의 역 이름과 도착역의 역 이름이 바뀌는 기능
**좌석 예매 시 예약 확인에서 확인 기능**

- 홈화면에서 바텀앱바의 오른쪽에 있는 티켓 모양을 누르면 예매 확인 페이지로 이동
- 예매 확인 페이지에서 내가 예매한 티켓들을 모두 볼 수 있음
## 구성하기

### 컴포넌트화

- 레이아웃
  - HomePage ⇒ 전체적으로 세로로 배치(Column 사용)
  - TrainListPage ⇒ 전체적으로 세로로 배치(Column 사용)
  - SeatPage ⇒ 전체적으로 세로로 배치(Column 사용)
  - MyBookingsPage ⇒ 전체적으로 세로로 배치(Column 사용)
- 나누어진 레이아웃 별 필요한 요소 구상하기
**HomePage**

  - 최상단 ⇒ AppBar
  - 출발역, 도착역 역 선택 ⇒ selectStationBox
  - 좌석 선택 버튼 ⇒ selectSeatButton
  - 최하단 ⇒ BottomAppBar
**TrainListPage**

  - 최상단 ⇒ AppBar
  - 역 표시 리스트 ⇒ stationList
**SeatPage**

  - 최상단 ⇒ AppBar
  - 출발역, 도착역 표시 ⇒ SeatPage
  - 좌석 선택 여부 표시 ⇒ SeatPage
  - 선택될 좌석 행 레이블 ⇒ SeatList
  - 선택될 좌석 리스트 및 레이블 ⇒ SeatList
  - 좌석 예매하기 버튼 ⇒ bottomButton
  - 최하단 ⇒ BottomAppBar
**MybookingsPage**

  - 최상단 ⇒ AppBar
  - 예매한 티켓 확인 ⇒ BookingsList
  - 최하단 ⇒ BottomAppBar
- 요소 별 필요한 위젯 구상하기
**HomePage**

  - AppBar ⇒ Text
  - selectStationBox ⇒ Row(Column(Text,Text), Container, Column(Text,Text))
  - selectSeatButton ⇒ ElevatedButton
  - BottomAppBar ⇒ Row(Icon, Icon)
**TrainListPage**

  - AppBar ⇒ Text
  - stationList ⇒ Container(Text)
**SeatPage**

  - AppBar ⇒ Text
  - SeatPage ⇒ Row(Text, Icon, Text)
  - SeatPage ⇒ Row(Container, Text, Container, Text)
  - SeatList ⇒ Row(Container(Text), Container(Text), Container(Text), Container(Text))
  - SeatList ⇒ Row(Container, Container, Text, Container, Container)
  - bottomButton ⇒ Container(Stack(SafeArea(ElevatedButton)))
  - BottomAppBar ⇒ Row(Icon, Icon)
**MybookingsPage**

  - AppBar ⇒ Text
  - BookingsList ⇒ 
Row(Container(Row(Text, Icon, Text)), Container, Container(Column(Text, Text)))
  - BottomAppBar ⇒ Row(Icon, Icon)
- 디렉토리 분리
```dart
📁 lib/
├── main.dart
├── theme.dart
├── 📁 model/
│    └── ticket.dart
├── 📁 home/
│    ├── home_page.dart
│    └── 📁 widgets/
│         ├── select_station_box.dart
│         └── select_seat_button.dart
├── 📁 train_list/
│    ├── train_list_page.dart
│    └── 📁 widgets/
│         └── station-list.dart
├── 📁 seat/
│    ├── seat_page.dart
│    └── 📁 widgets/
│         ├── seat_list.dart
│         └── bottom_button.dart
└── 📁 bookings/
     ├── my_bookings_page.dart
     └── 📁 widgets/
          └── bookings_list.dart

```

### 상태 관리 구성하기

- HomePage
**역 선택**

  - HomePage(출발역/도착역) → train_list_Page(appBar)
  - train_list_Page(역선택) → HomePage(선택한 역 표시 후 다시 빌드)
- Train_List_Page
**역 리스트**

  - homePage에서 출발역/도착역 글자 정보만 받아와서 빌드
  - 선택된 역이 있으면 제외하고 다시 재빌드
- Seat_Page
**좌석 선택**

  - 좌석 선택 후 seat_Page 재빌드 ⇒ seat-page Stateful위젯으로 변환
  - 선택된 좌석을 행과 열로 좌표화 하여 선택된 좌표 색 변화 후 재빌드
  - 이후 선택된 좌석 좌표 showCupertinoDialog에 정보 보내기
## 구현 완료

- 구현 완료 
## 트러블슈팅 과정

**Vertical viewport was given unbounded height 에러**

- 문제 : ListView를 썼더니 unbounded height 에러가 발생함
- 원인 : ListView는 스크롤 가능한 위젯이라 스크롤이 가능한 높이를 알아야 하는데 부모가 높이 제한을 주지 않아서 발생함
- 해결 : Expanded를 사용하여 영역에 대한 높이를 명확히 해줌 
```dart
Column(
  children: [
    Text('제목'),
    Expanded( // ✅ ListView에 공간을 명확히 줌
      child: ListView(children: [...]),
    ),
  ],
)
```

**리스트 children 안에서 for문으로 반복하기**

- 문제 : for문으로 SeatList()를 반복하려고 했는데 에러 발생
- 원인 : children 안에서 for문으로 SeatList를 하나씩 꺼내려고 하였으나, 이미 children이라는 리스트 안에 들어있기 때문에 리스트 안에서 리스트를 넣는 것은 불가능함
- 해결 : ...[] (spread operator)를 사용하여 children 리스트 안에 펼쳐서 꺼내 놓음
```dart
children: [
  for (int i = 1; i <= 20; i++) ...[
    SeatList('$i'),
    if (i != 20) SizedBox(height: 8),
  ]
]
```

**SafeArea 위치와 사용 방식**

- 문제 : SafeArea를 사용했는데, 시스템 UI 공간보다 더 크게 보호되는 것 같음
- 원인 : SafeArea는top, bottom으로 크기 조절이 가능하지만, 최소 보호값이 있어 크기를 줄일 수 없음
- 해결 : 최소 크기만큼 줄이고 넘어감
**Stack 구조**

- 문제 : Navigator.push를 사용하여 다시 홈 화면으로 이동하지만 상태가 변경되지 않음
- 원인 : push를 사용하게 되면 스택 구조가 [홈 - 페이지 1 - 홈] 구조로 다시 홈이 빌드 되어 상태가 변경 되지 않음
- 해결 : 결과 값을 가지고 이전 홈 화면으로 돌아가려면 Navigator.pop을 사용해야 함
**예약된 좌석의 출발역, 도착역, 좌표를 저장하기**

- 문제 : 예약되는 좌석의 출발역, 도착역, 좌표를 저장하기 위해 파라미터를 넘기는 방법이 복잡
- 원인 : 출발역, 도착역 선택은 homePage에 있으며 좌석 좌표는 seatPage에 존재하여 파라미터로 넘겨주기엔 너무 복잡함
- 해결 : 
  - 새로운 모델 파일에 Ticket 클래스를 만들고 역, 좌표를 파라미터로 가져와 저장
```dart
class Ticket {
  Ticket(this.arrStation, this.depStation, this.rowNum,
      this.colNum);
  String arrStation;
  String depStation;
  String rowNum;
  int colNum;
}
```

  - async와 await를 사용하여 좌석 예매 버튼을 눌렀을 때 실행이 끝나지 않고 결과 값을 가져와서 다시 진행하는 비동기로 진행 
```dart
onPressed: () async {
         if (selectedRow == null &&
             selectedCol == null) {
         } else {
           bool? result =                    // 예매확인이 끝나고 결과를 받음
               await showCupertinoDialog(    // 비동기 시작
               .
               .
               .
            if (result == true) {            // 결과 값에 따라 내용 저장
             Navigator.pop(
                 context,                         // 이전 페이지로 이동하며
                 Ticket(arrStation, depStation,   // Ticket에 결과값을 반환  
                     selectedRow!, selectedCol!));
           }   
```

  - Ticket에 결과 데이터를 가진 result를 받음
```dart
onPressed: () async {
            if (arrivalStation != '선택' &&
                departureStation != '선택') {   // 비동기로 코드 완료 후 실행 
final result = await Navigator.push(   // final로 완료된 코드의 result를 받음
                context,
                MaterialPageRoute(
                  builder: (context) {
                    return SeatPage(
                        arrivalStation, departureStation);
                  },
                ),
              );
              if (result != null) {  // 이 result의 값이 있으면 함수 실행
                onbooked(result);
              }
             }
```

  - 함수를 사용하여 result의 데이터 값을 리스트화 시키고 리스트를 예약 확인 페이지로 보냄
```dart
List<Ticket> ticketList = [];

  void onBooked(Ticket ticket) {
    ticketList.add(ticket);          // Ticket 타입 데이터를 리스트에 추가
  }
  .
  .
  .
  IconButton(
   onPressed: () {
     Navigator.pushNamed(
         context, '/myBookingsPage',
         arguments: ticketList);        // 이 리스트를 arguments로 보냄
      },
      icon: Icon(Icons.confirmation_num))
```

  - 예약 확인 페이지에서 데이터를 받고 해당되는 값에 넣어줌 
```dart
class MyBookingsPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final args = ModalRoute.of(context)!.settings.arguments
        as List<Ticket>;  // ticketList를 args에 받음
    return Scaffold(
    body: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 5),
        child: Column(
          children: [
            for (var i = 0; i < args.length; i++) ...[
              BookingsList(i),
              if (i < args.length) SizedBox(height: 4)
            ]  // 저장된 예약 만큼 풀어서 위젯으로 구성
          ],
        ),
      ),
```
