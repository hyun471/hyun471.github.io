---
title: "assignment 5"
status: "published"
date: "2026-04-04"
type: "posts"
---

# assignment 5

# 📚 영화 정보 앱

## 영화 정보 앱

![](/images/32_assignment 5/img_01.png)

![](/images/32_assignment 5/img_02.png)

![](/images/32_assignment 5/img_03.png)

![](/images/32_assignment 5/img_04.png)

![](/images/32_assignment 5/img_05.png)

### 과제 시나리오

- 목표 : TMDB API와 연동하여 데이터를 Hero 애니매이션을 활용하여 클린 아키텍쳐 구조로 만들기
**기본 조건**

**필수 기능 :**

- 기본 다크 모드이며 라이트 모드는 지원 안함
- 영화 리스트는 가로방향 리스트 뷰이며 이미지는 20개씩 출력
- 인기순 이미지 출력시 랭킹 숫자는 이미지 위 좌측 하단에 출력
- 디테일 페이지는 영화제목, 개봉일, 태그라인, 러닝타임, 카테고리, 영화설명, 평점, 평점투표 수 , 인기 점수, 예산, 수익, 제작사 로고를 표시
- Hero 위젯을 이용하여 디테일 페이지로 전환 시 포스터에 애니메이션 효과를 적용
- 클린 아키텍쳐 + MVVM 구조 생성
**도전 :**

- 클린 아키텍쳐 구현한 각 단계별 테스트 코드 작성
- 인기순 영화 리스트에서 마지막에 위치했을 때 다음 순위 영화들을 불러오기
- 영화 전체 목록을 보여주는 페이지에서 스크롤을 아래로 당겼을 때 영화 목록을 새로 불러옴
**UI 디자인 명세 :**

**HomePage**

[body 영역] : 다크모드

1. [가장 인기있는] :
  - 영화 포스터 
  - 모서리 둥글기
  - Hero 애니메이션으로 연결
1. [현재 상영중] :
  - 영화 포스터 리스트 : ListVeiw
  - 모서리 둥글기
  - Hero 애니메이션으로 연결
1. [인기 순] :
  - 영화 포스터 리스트 : ListVeiw
  - 인기순 숫자 : 크기 80
  - 모서리 둥글기
  - Hero 애니메이션으로 연결
1. [평점 높은순] :
  - 영화 포스터 리스트 : ListVeiw
  - 모서리 둥글기
  - Hero 애니메이션으로 연결
1. [개봉예정] :
  - 영화 포스터 리스트 : ListVeiw
  - 모서리 둥글기
  - Hero 애니메이션으로 연결
**ReviewPage**

1. [영화 포스터] ⇒ 모서리 둥글기 12, Hero 애니메이션으로 연결
1. [영화 제목] ⇒ 글자 크기 24
1. [개봉일, 한줄 요약, 상영시간] ⇒ 글자 크기 기본
1. [장르] ⇒ 박스 높이 : 32, 박스 테두리 색 : grey[800], 글자 크기 : 기본, 글자 색 : blueAccent
1. [줄거리] ⇒ 글자 크기 기본, 글자 수만큼 높이 차지
1. [흥행정보] ⇒ 박스 높이 : 70, 테두리 색 : grey[800]
1. [제작사 로고] ⇒ 박스 높이 : 80, 박스 색 : grey[300], 패딩 : 전체 15
## 구성하기

### 컴포넌트화

- 레이아웃
  - HomePage ⇒ 전체적으로 세로로 스크롤 가능하게 배치(ListView 사용)
  - DetailPage ⇒ 전체적으로 세로로 스크롤 가능하게 배치(ListView 사용)
- 나누어진 레이아웃 별 필요한 요소 구상하기
**HomePage**

  - 가장 인기있는 영화 포스터 ⇒ MostPopular
  - 영화 리스트에 작은 포스터 ⇒ ThmbnailBox
  - 영화 리스트 ⇒ MovieList
  - 인기 순 영화 리스트 ⇒ PopularWidget
**DetailPage**

  - 영화 포스터 ⇒ Hero(Image)
  - 장르 박스 ⇒ Category
  - 흥행정보 박스 ⇒ BoxOfficeInfo
  - 제작사 로고 ⇒ CompaniesLogo
- 요소 별 필요한 위젯 구상하기
**HomePage**

  - MostPopular ⇒ Hero(SizedBox(Image))
  - ThmbnailBox ⇒ Hero(SizedBox(Image))
  - MovieList ⇒ ListView(ThumbnailBox)
  - PopularWidget ⇒ ListView(SizedBox(Stack(ThumbnailBox, Text)))
**DetailPage**

  - Category ⇒ Column(ListView(Container(Text)))
  - BoxOfficeInfo ⇒ ListView(Container(Column(Text, Text)))
  - CompaniesLogo ⇒ ListView(Container(Image))
**디렉토리 분리**

```dart
📁 lib/
├── main.dart
├── 📁 data/
│    ├── 📁 data_source/
│    │    ├── movie_list_data_source_Impl.dart
│    │    └── movie_list_data_source.dart
│    ├── 📁 dto/
│    │    ├── moive_list_dto.dart
│    │    └── movie_detail_dto.dart
│    └── 📁 repository/
│         └── movie_poster_repository_impl.dart
├── 📁 domain/
│    ├── 📁 entitiy/
│    │    ├── movie_detail.dart
│    │    └── movie_poster.dart
│    ├── 📁 repository/
│    │    └── movie_poster_repository.dart
│    └── 📁 usecase/
│         └── fetch_movie_poster_usecase.dart
└── 📁 presentation/
     ├── provider.dart 
     └── 📁 page/
          ├── 📁 home/
          │    ├── home_page.dart
          │    ├── 📁 view_model/
          │    └── 📁 views/
          └── 📁 detail/
               ├── detail_page.dart
               ├── 📁 view_model/
               └── 📁 views/
```

### Riverpod 상태 관리 구성하기

- 클린 아키텍쳐
**클린 아키텍쳐**

  - Provider를 통해 MovieListDataSource 참조하여 MovieListDataSource 구현체 객체 생성
  -  구현체 객체에서 TMDB API를 통해 모든 데이터를 각각 Dto 모델 타입으로 모델로 저장
  -  MoviePosterRepository를 참조하여 저장된 데이터를 가공하고 구현체로 보내며 객체 생성
  - 구현체에서 현재 앱에서 필요한 정보만을 각각 Entity 모델 타입으로 모델 저장
  - 이후 이 데이터를 가공하여 FetchMoviePosterUsecase 보내며 객체 생성
- Home_Page
** Home_Page**

  - 클린 아키텍쳐에서 생성된 객체로 영화 리스트를 불러오는 메서드를 사용하여 상태를 불러옴
  - 현재 상영중, 인기순, 평점 높은순, 개봉예정 총 4개의 상태를 불러와 홈 뷰 모델 상태에 각각 저장
  - 이후 상태들을 UI에 적용
- detail_Page
**리뷰 리스트 박스**

  - 클린 아키텍쳐에서 생성된 객체로 영화 리스트를 불러오는 메서드를 사용하여 상태를 불러옴
  - 불러온 상태를 상세 영화 정보 뷰모델 상태에 저장
  - 이후 상태를 UI에 적용
## 구현 완료

- 구현 완료 
## 트러블슈팅 과정

**뷰 모델에서 여러개의 상태를 관리**

- 문제 : 뷰모델에서 여러개의 상태를 관리할 때 효과적인 방법을 구상
- 원인 :
  -  homeViewModel에서 4개의 상태를 적용하기 위해 다음과 같이 작성하면 하나의 메서드에 대한 결과를 넣어주면 덮어쓰기가 됨, 즉 NowPlaying에 대한 메서드가 실행되면 나머지 3개의 상태는 빈 리스트로 값이 바뀜
**코드**

```dart
// 상태 클래스
class HomeState {
  HomeState({
    required this.nowPlaying,
    required this.popular,
    required this.topRated,
    required this.upComing,
  });
  List<MoivePoster> nowPlaying;
  List<MoivePoster> popular;
  List<MoivePoster> topRated;
  List<MoivePoster> upComing;
 }
 
 // 뷰 모델
 class HomeViewModel extends Notifier<HomeState> {

   @override
  HomeState build() {
    fetchNowPlaying();
    fetchPopular();
    fetchTopRated();
    fetchUpComing();
    return HomeState(
      nowPlaying: [],
      popular: [],
      topRated: [],
      upComing: [],
    );
  }

  Future<void> fetchNowPlaying() async {
    final nowPlayingRepo = ref.read(fetchMoviesUsecaseProvider);
    final result = await nowPlayingRepo.getNowPlayingMovie(1);
    state = HomeState(
      nowPlaying: result,
      popular: [],
      topRated: [],
      upComing: [],
    );
  }
 }
```

- 해결 : 
  - 상태 클래스에서 copyWith를 만들어서 상태를 하나 씩 불러와 적용할 수 있게 변경
**코드 **

```dart
// 상태 클래스
class HomeState {
  HomeState({
    required this.nowPlaying,
    required this.popular,
    required this.topRated,
    required this.upComing,
  });
  List<MoivePoster> nowPlaying;
  List<MoivePoster> popular;
  List<MoivePoster> topRated;
  List<MoivePoster> upComing;

  HomeState copyWith({
    List<MoivePoster>? nowPlaying,
    List<MoivePoster>? popular,
    List<MoivePoster>? topRated,
    List<MoivePoster>? upComing,
  }) {
    return HomeState(
      nowPlaying: nowPlaying ?? this.nowPlaying,
      popular: popular ?? this.popular,
      topRated: topRated ?? this.topRated,
      upComing: upComing ?? this.upComing,
    );
  }
}

// 뷰 모델
class HomeViewModel extends Notifier<HomeState> {

  @override
  HomeState build() {
    fetchNowPlaying();
    fetchPopular();
    fetchTopRated();
    fetchUpComing();
    return HomeState(
      nowPlaying: [],
      popular: [],
      topRated: [],
      upComing: [],
    );
  }

  Future<void> fetchNowPlaying() async {
    final nowPlayingRepo = ref.read(fetchMoviesUsecaseProvider);
    final result = await nowPlayingRepo.getNowPlayingMovie(1);
    state = state.copyWith(nowPlaying: result);
  }

  Future<void> fetchPopular() async {
    final popularRepo = ref.read(fetchMoviesUsecaseProvider);
    final result = await popularRepo.getPopularMovie(1);
    state = state.copyWith(popular: result);
  }

  Future<void> fetchTopRated() async {
    final topRatedRepo = ref.read(fetchMoviesUsecaseProvider);
    final result = await topRatedRepo.getTopRatedMovie(1);
    state = state.copyWith(topRated: result);
  }

  Future<void> fetchUpComing() async {
    final upComingRepo = ref.read(fetchMoviesUsecaseProvider);
    final result = await upComingRepo.getUpcomingMovie(1);
    state = state.copyWith(upComing: result);
  }
```

**상태를 UI에 적용 중 StateError (Bad state: No element) 오류 발생**

- 문제 : UI에서 상태에 대한 값을 받아 UI에 적용해야 하지만 상태에서 빈 리스트가 넘어와 에러가 발생
- 원인 : 
  - 처음 앱을 실행하여 UI를 빌드 될 때 UI가 먼저 빌드되고 상태를 가져오게 됨
  - 따라서 처음 먼저 빌드될 때 상태에 데이터가 없는 상태이기 때문에 데이터가 없으면 빈 리스트를 반환하게 되어 있어 빈 리스트가 반환되어 오류가 발생
- 해결 : 
  - 상태에 데이터가 비어있으면 로딩이 표시되고 상태에 데이터가 존재하면 UI를 로드 하도록 변경
**코드**

```dart
        Scaffold(
      body: 
        child: Padding(
          padding: const EdgeInsets.all(15),
          child:
              // 삼항 연산자를 이용하여 상태가 하나라도 비어있으면
              // 로딩 위젯이 표시되고 모두 비어있지 않으면 위젯을 빌드
              state.nowPlaying.isEmpty ||
                  state.popular.isEmpty ||
                  state.topRated.isEmpty ||
                  state.upComing.isEmpty
              ? Center(child: CircularProgressIndicator())
              : ListView(
                  children: [
                  ...
```

# 📝 치트시트

## 📕 New 위젯 정리 :

## 📒 New 용어 정리
