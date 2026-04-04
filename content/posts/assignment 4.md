---
title: "assignment 4"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 지역 검색 앱

## 지역 검색 앱

![](/images/31_assignment 4/img_01.png)

![](/images/31_assignment 4/img_02.png)

![](/images/31_assignment 4/img_03.png)

### 과제 시나리오

- 목표 : 지역 검색 API와 서버 연동하여 상태 관리하기
**기본 조건**

**필수 기능 :**

- AppBar에 TextField 위젯 배치
- 네이버 검색 API 발급 후 Location모델 클래스 만들기
- LocationRepository클래스 생성 후 dio패키지 사용하여 검색 키워드 구현
- flutter_riverpod패키지를 사용하여 상태 관리
- Firestore 데이터 베이스 활성화하고 reviews라는 컬렉션 만들기
- Firestore 사용을 위한 firebase_core, cloud_firestore 패키지 추가
- 리뷰 내용을 저장할 모델 클래스 만들기
- 리뷰를 저장하는 메서드와 mapX, mapY 좌표에 해당하는 리뷰를 불러오는 메서드 구현
**도전 :**

- 현재 디바이스의 위치를 vworld API로  주소를 조회한 뒤 네이버 API로 검색하여 뷰모델에 요청
- 나만의 기능 : 리뷰 작성 시 패스워드 설정, 리뷰 삭제 시 패스워드가 일치하면 삭제
**UI 디자인 명세 :**

**HomePage**

1. [앱바] : 
  - TextField : 테두리 ⇒ 모든 테두리, 모서리 둥글기 ⇒  ,
1. [body 영역] :
  - background 색상 ⇒ 연분홍
  - body내의 위젯들은 세로 가운데 정렬
1. [지역 박스] :
  - 높이 ⇒  , 색상 ⇒ Colors.white, 모서리 둥글기 ⇒  
1. [지역 박스 글자] :
  - 지역명 : 크기 ⇒   , 두께 ⇒ FontWeight.bold
  - 상세 지역명 : 크기 ⇒
**ReviewPage**

1. [앱바] : 선택된 지역명, 크기 ⇒  
1. [뒤로가기 버튼] : 별도로 선언하지 않고 APPBar에서 기본으로 나오는 버튼 사용
1. [body 영역] : 
  - background 색상 ⇒ Colors.white
1. [Review 내용 박스] : 높이 ⇒  , 색상 ⇒ Colors.white, 모서리 둥글기 ⇒  
1. [Review 내용 글씨] : 
  - Review 내용 : 크기 ⇒   , 두께 ⇒ FontWeight.bold
  - 작성 시간 : 크기 ⇒  , 색상 ⇒  
1. [Review 작성 박스] : 
  - 하단에 위치
  - 작성 박스 : 크기 ⇒  , 색상 ⇒  , 모서리 둥글기 ⇒ 위쪽만  
  - TextField : 테두리 ⇒ 아래쪽만, 테두리 색상 ⇒  
## 구성하기

### 컴포넌트화

- 레이아웃
  - HomePage ⇒ 전체적으로 세로로 배치(Column 사용)
  - ReviewPage ⇒ 전체적으로 세로로 배치(Column 사용)
- 나누어진 레이아웃 별 필요한 요소 구상하기
**HomePage**

  - 최상단 ⇒ AppBar
  - 지역 박스 ⇒ localBox
**ReviewPage**

  - 최상단 ⇒ AppBar
  - 리뷰 박스 ⇒ ReviewBox
  - 최하단 ⇒ addReviewBox
- 요소 별 필요한 위젯 구상하기
**HomePage**

  - AppBar ⇒ TextField
  - localBox ⇒ Container(Column(Text, Text))
**ReviewPage**

  - AppBar ⇒ TextField
  - ReviewBox ⇒ Container(Column(Text, Text))
  - addReviewBox ⇒ Container(TextField)
- 디렉토리 분리
```dart
📁 lib/
├── main.dart
├── 📁 commons/
│    ├── 📁 models/
│    │    ├── local_model.dart
│    │    └── review_model.dart
│    └── 📁 repository/
│         ├── location_repository.dart
│         ├── my_location_from_device.dart
│         ├── my_location_repository.dart
│         └── review_repository.dart
└── 📁 pages/
     ├── 📁 home/
     │    ├── home_page.dart
     │    ├── 📁 views/
     │    │    └── local_info_box.dart   
     │    └── 📁 viewmodels/
     │         └── home_view_model.dart
     └── 📁 review/
          ├── review_page.dart
          ├── 📁 views/
          │    ├── review_box.dart
          │    └── write_review_box.dart
          └── 📁 viewmodels/
               └── review_view_model.dart
```

### Riverpod 상태 관리 구성하기

- Home_Page
**지역명 리스트 박스**

  - 네이버 API에서 받은 지역 데이터를 local모델 클래스 타입으로 저장
  - 뷰모델에 지역 검색 후 상태에 저장하는 메서드 구성
  - 뷰 모델에 현재 위치를 vworld API에서 지역명 받아서 네이버 API로 지역 검색하는 메서드 구성
  - 필요한 UI에서 뷰모델을 호출하고 메서드를 사용하여 상태에 저장된 값을 UI에 사용
- review_Page
**리뷰 리스트 박스**

  - 리뷰가 작성된 지역의 리뷰를 서버에서 불러와 review모델 클래스 타입으로 저장
  - 뷰 모델에서 review모델 클래스 타입의 데이터를 상태에 저장
  - 뷰 모델에서 review모델 클래스 타입을 서버에 저장하는 메서드 구성
  - 뷰 모델에서 서버에 있는 리뷰 데이터를 삭제하는 메서드 구성
  - 필요한 UI에서 뷰모델을 호출하고 메서드를 사용하여 상태를 불러와 UI에 적용, 변경된 상태를 서버에 저장, 서버에 있는 데이터 삭제 후 UI에 반영을 UI에 사용
## 구현 완료

- 구현 완료 
## 트러블슈팅 과정

**flutter pub add riverpod문제**

- 문제 : 상태관리를 하기 위하여 ConsumerStateFulWidget이나 consumerWidget을 사용하려 했으나 인식되지 않음
- 원인 : flutter pub add riverpod만 입력하면 프로젝트에는 추가되지만, flutter_riverpod패키지가 아닌 riverpod만 설치됨. UI에서 사용하는 ConsumerWidget계열은 flutter_riverpod패키지에 존재
- 해결 : 터미널에서 flutter_riverpod패키지를 설치
```dart
flutter pub add flutter_riverpod
```

**API에서 받은 지역명을 Reviw클래스 title에 넘기는 방법**

- 문제 : API에서 받아온 title을 Review 클래스의 title로 할당해야 함
- 원인 : API에서 받아온 데이터의 title를 파라미터로 받지 않아 할당할 수 없음
- 해결 : ReviewRepository.insert() 호출 시 title 파라미터에 전달해야 함
**firebase 데이터베이스에 지역에 따른 리뷰를 저장하는 방식**

- 문제 : 서버에 지역에 따른 리뷰를 저장하기 위하여 지역에 대한 컬렉션을 만들고 그 안에 리뷰에 대한 컬렉션을 만드려고 함
- 원인 : 컬렉션 안에 컬렉션을 만들어도 되지만 구조가 복잡해짐
- 해결 : 
  - 리뷰에 대한 컬렉션만 만들고 지역에 대한 좌표를 같이 저장
```dart
reviews (컬렉션)
  └─ {자동 문서ID}
        ├─ title: "지역 이름"
        ├─ content: "리뷰 내용"
        ├─ mapX: "좌표 X"
        ├─ mapY: "좌표 Y"
        └─ date: Timestamp
```

  - 해당 지역을 선택했을 때 .where('mapX', isEqualTo: x)와.where('mapY', isEqualTo: y)으로해당 지역의 좌표값과 일치하는 리뷰들만 불러오도록 구현 
```dart
  // 좌표를 활용하여 해당 좌표의 모든 리뷰 불러오기
  Future<List<Review>> reviewStream({
    required String mapX,
    required String mapY,
  }) async {
    final firestore = FirebaseFirestore.instance;
    final collectionRef = firestore.collection('reviews');
    final result = await collectionRef
        .where(
          'mapX',
          isEqualTo: mapX,
        )
        .where(
          'mapY',
          isEqualTo: mapY,
        )
        .get();
    final streamReviews = result.docs.map(
      (doc) {
        return Review.fromJson(
          doc.data(),
          doc.id,
        );
      },
    ).toList();
    return streamReviews;
  }
```

**리뷰 갱신 무한 루프로 인한 앱이 꺼지는 현상**

- 문제 : 에뮬레이터에서 앱 실행 시 ReviewPage가 켜지자마자 멈추고 종료됨
- 원인 : 
  - ReviewViewModel의 build에서 getAllReviews()를 바로 호출하여 무한 루프 발생
  - 내부에서 Stream.listen() 상태를 갱신과 Riverpod ref.watch를 사용하여 무한 루프 발생
```dart
// 뷰모델
class ReviewViewModel
    extends AutoDisposeFamilyNotifier<ReviewState, Local> {
  @override
  ReviewState build(local) {
    getAllReviews(local: local);
    return ReviewState([]);
  }

  // 리뷰 갱신
  void getAllReviews({required Local local}) async {
    final reviewRepository = ReviewRepository();
    final reviews = await reviewRepository.reviewStream(
      mapX: local.mapX,
      mapY: local.mapY,
    );
    final streamSubscription = reviews.listen((event) {
      state = ReviewState(event);
    });

    ref.onDispose(() {
      streamSubscription.cancel();
    });
  }
}
```

- 해결 : 
  - build에서 getAllReviews()를 바로 호출을 삭제 ← 무한루프의 주 원인
  - Stream.listen을 제거하고, ref.watch로 상태만 구독
  - get()으로 한 번만 불러와 상태 관리
**날짜 순으로 리뷰를 정렬 시 Firestore 인덱스 문제**

- 문제 : 해당 지역 좌표를 불러와 데이터를 정렬하는 .where(mapX).where(mapY).orderBy(date)를 사용하였을 때 FirebaseException [failed-precondition] The query requires an index. 오류 발생
- 원인 : Firestore에서 여러 조건(where + orderBy)을 같이 쓰면 복합 인덱스가 필요
- 해결 : 
  - 1. Firebase 서버에서 database에 색인에서 복합 인덱스를 생성하여 정렬 기준을 입력해 사용이 가능 (코드 내 복잡도 완화)
  - 2. flutter 코드에서 데이터를 받아와 sort라는 메서드로 데이터를 정렬할 수 있음
  - 데이터가 많고 복잡한 경우 색인을 사용하여 복합 인덱스를 만드는 것이 좋으나 정렬 기준이 작고 데이터가 많지 않으면 코드에서 정렬하는 것도 좋음
```dart
// 데이터를 받아와서 List.sort()로 정렬
  Future<List<Review>> reviewStream({
    required String mapX,
    required String mapY,
  }) async {
    final firestore = FirebaseFirestore.instance;
    final collectionRef = firestore.collection('reviews');
    final result = await collectionRef
        .where(
          'mapX',
          isEqualTo: mapX,
        )
        .where(
          'mapY',
          isEqualTo: mapY,
        )
        .get();
    final streamReviews = result.docs.map(
      (doc) {
        return Review.fromJson(
          doc.data(),
          doc.id,
        );
      },
    ).toList();
    streamReviews.sort((a, b) => b.date.compareTo(a.date));
    return streamReviews;
  }
```

**날짜 저장 시 서버에는 Timestamp타입으로 저장되어 타입 오류 발생**

- 문제 : 서버에 현재 시간을 저장하면 타입 불일치 오류 발생
- 원인 : 서버에 DateTime.now()을 바로 저장하면 Timestamp타입으로 저장됨. Review모델에서 String타입으로 받으면 타입 불일치 오류 발생
- 해결 : 
  - 서버에 저장을 String타입으로 변환하고 저장해도 되지만 flutter에서 다룰 때는 시간을 DateTime타입으로 다루는게 더 효과적
  - 따라서 먼저 모델에서 타입을 DateTime으로 변경하고 서버에서 데이터를 받아올 때 Timestamp타입에서 DateTime으로 변경 
```dart
class Review {
  String id;
  String title;
  String content;
  String mapX;
  String mapY;
  DateTime date;
  String password;
  Review(
      {required this.id,
      required this.title,
      required this.content,
      required this.date,
      required this.mapX,
      required this.mapY,
      required this.password});

  Review.fromJson(Map<String, dynamic> map, String id)
      : this(
            id: id,
            title: map['title'],
            content: map['content'],
            date: (map['date'] as Timestamp).toDate(),
            mapX: map['mapX'],
            mapY: map['mapY'],
            password: map['password']);
```

  - UI에서 표시할 때는 DateTime을 String타입으로 변경
```dart
                Text(
                  review.date.toIso8601String(),
                  style: TextStyle(
                    fontSize: 14,
                    color: Colors.grey,
                  ),
                ),
```

**주소명이 길어질 경우 Overflow나옴**

- 문제 : 긴 주소명을 넣었더니 Overflow에러 발생
- 원인 : 컨테이너의 크기가 고정되어 있어 Text위젯의 글자가 길어지면 한 줄에서 벗어나서 줄바꿈이 진행되어 overflow가 발생함
- 해결 : Container의 최소,최대 크기를 지정하는 constraints라는 속성을 사용 
```dart
      child: Container(
        constraints: BoxConstraints(minHeight: 110), // 최소 높이 110
        width: double.infinity,
        decoration: BoxDecoration(
            color: Colors.white,
            borderRadius: BorderRadius.circular(20),
            border: Border.all(
              color: Color(0xffEFECEF),
              width: 2,
            )),
```

**현위치 아이콘 클릭 시 앱이 다운되는 현상**

- 문제 : 현위치 버튼 클릭 시 앱이 종료됨
- 원인 : 
  - 디바이스의 현재 좌표가 미국으로 되어 있어 국내에서 쓰는 API가 사용되지 못함
  - 디바이스의 현재 좌표를 서울로 변경 시 API에 보내는 좌표의 정확도 차이가 발생하여 API에서 데이터를 뽑아내지 못함
- 해결 : 
  - vworld의 Geocoder API 2.0를 사용하면 정밀한 좌표를 보내야 정상적인 데이터를 받을 수 있음
  - vworld의 2D데이터 API 2.0에서 읍면동으로 검색하는 API를 사용하면 낮은 정확도를 가진 좌표로도 검색이 가능하여 2D데이터 API 2.0에 읍면동 검색 API를 사용