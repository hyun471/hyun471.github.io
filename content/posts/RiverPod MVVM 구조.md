---
title: "RiverPod MVVM 구조"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 RiverPod 상태관리

- Provider / riverpod 상태 관리 정리 링크
## RiverPod 상태관리 MVVM 구조 흐름

- MVVM 디렉토리 구조
**상태 관리 흐름**

- Model 구성
  - Model 클래스 구성
  - Repository(Service) 클래스 구성
- ViewModel 구성
  - 상태 클래스 구성
  - ViewModel(Notifier를 상속한) 구성
  - Provider 등록
- View(UI)에서 Consumer를 사용하여 상태 관리
### Model (서버, API)

- 서버나 API 등에서 데이터를 받아 dart언어로 구성하는 곳
- 서버 연동 링크 (API 및 Firebase서버 연동)
**앱에서 다뤄지는 데이터의 model 구성**

- 앱에서 다뤄지는 데이터 값들을 클래스로 구성
```dart
class User {
  String name;
  int age;

  User({required this.name, required this.age});
}
```

- 클래스 안에 fromJson 네임드 생성자 만들기 (어떤 항목에 어떤 데이터를 넣을지 정함)
```dart
User.fromJson(Map<String, dynamic> map)
		: this(
					name: map['name'],
					age: map['age'],
			);
```

- 클래스 안에 toJson 메서드 만들기 (어떤 항목의 값을 어떤 데이터로 보낼지 정함)
```dart
Map<String, dynamic> toJson() {
	return {
		'name': name,
		'age': age,
	}
}
```

**서버나 API 등에서 받아오는 데이터의 model 구성 및 메서드 만들기 (주로 Repository등의 폴더에 구성)**

**서버나 API 등에서 데이터 불러오는 메서드 만들기 (데이터 가져오는 과정은 비동기로 진행)**

**1. API에서 데이터 받고 가공하기**

- API에 데이터 요청하기
  - 사용할 API의 사용법을 숙지하고 사용
```dart
class BookRepository {
  Future<List<Book>> searchbooks(String query) async {
    // HTTP 패키지에 있는 Http를 요청하는 Client 사용하여 객체화
    final client = Client();
    // HTTP 패키지에 get을 사용하여 Uri와 헤더를 보내 해당 정보 요청
    final response = await client.get(
	    // 문자열 주소를 Uri 타입으로 변환하여 보냄 (필수)
      Uri.parse(
        'https://openapi.naver.com/v1/search/book.json?query=$query',
      ),
      // 어떤 곳에서 요청했는지 알 수 있는 ID와 Secret 넣기 (필수)
      headers: {
        'X-Naver-Client-Id': '여기에 ID',
        'X-Naver-Client-Secret': '여기에 Secret',
      },
    );
```

**2. 서버(Firebase)에서 데이터 받고 가공하기**

- 서버(Firebase)에 데이터 요청하기
```dart
class PostRepository {
  Future<List<Post>?> getAll() async {
    try {
      // 1. 파이어스토어 인스턴스 가지고 오기
      final firestore = FirebaseFirestore.instance;
      // 2. 컬렉션 참조 만들기
      final collectionRef = firestore.collection('posts');
      // 3. 값 불러오기
      final result = await collectionRef.get();
```

**JSON → Map 변환 (jsonDecode)**

- API에서 받은 데이터 변환 
```dart
// 변환한 데이터 Map 타입의 map에 저장
Map<String, dynamic> map = jsonDecode(response.body);
// JSON에서 필요한 'item'에 데이터가 리스트로 구성되어 리스트로 뽑아서 저장
final items = List.from(map['items']);
```

- Firebase의 경우 데이터를 Map타입으로 받아서 변환이 필요 없음
**Map → Model 클래스로 변환 (fromJson)**

**API의 데이터 **

- Map → Model 객체로 변환
```dart
// 리스트의 각 요소를 하나씩 꺼내 새로운 Iterable 타입으로 저장
final iterable = items.map((e) {
				// fromJson으로 객체화
        return Book.fromJson(e);
      });
      
      // Iterable 타입에 저장된 요소들을 리스트화
      final list = iterable.toList();
      return list;
```

**Firebase  데이터**

- Map → Model 객체로 변환
```dart
// 'posts'의 데이터가 저장된 result에 문서들(docs)의 데이터와 id를 꺼내서 저장
final docs = result.docs;
      return docs.map((doc) {
        // 문서의 데이터를 꺼내서 map에 저장
        final map = doc.data();
        // 문서의 id를 꺼냄
        doc.id;
        // 문서의 id를 Map타입으로 추가하고 map정보도 추가하여 newMap에 저장
        final newMap = {'id': doc.id, ...map};
        // 객체화
        return Post.fromJson(newMap);
        // 리스트화
      }).toList();
```

**예외 처리**

**API 경우**

- HTTP 상태 코드를 통하여 응답 처리 
```dart
    // 통신 코드가 200이면 정상 통신
    if (response.statusCode == 200) {
      Map<String, dynamic> map = jsonDecode(response.body);
      final items = List.from(map['items']);
      // MappedIterable 이라는 클래스에 함수를 넘겨줄테네
      // 나중에 요청하면, 그때 List(items)들을 하나씩 반복문을 돌면서
      // 내가 넘겨준 함수를 실행시켜서 새로운 리스트로 돌려줘!
      final iterable = items.map((e) {
        return Book.fromJson(e);
      });

      final list = iterable.toList();
      return list;
    }
    
    // 요청 실패 시 빈 리스트 반환
    return [];
```

- HTTP 상태 코드에 따른 오류 종류
**Firebase 경우**

- try catch 로 예외 처리 
```dart
    try {
      // 1. 파이어스토어 인스턴스 가지고 오기
      final firestore = FirebaseFirestore.instance;
      // 2. 컬렉션 참조 만들기
      final collectionRef = firestore.collection('posts');
      // 3. 값 불러오기
      final result = await collectionRef.get();

      final docs = result.docs;
      return docs.map((doc) {
        final map = doc.data();
        doc.id;
        final newMap = {'id': doc.id, ...map};
        return Post.fromJson(newMap);
      }).toList();
      // 오류가 발생되면 오류 출력하고 null 반환
    } catch (e) {
      print(e);
      return null;
    }
```

**받아온 데이터를 가공하는 C.R.U.D 등의 메서드 만들기**

**Firebase에서 사용하는 CRUD 메서드**

**Creat**

- 새로운 데이터를 서버에 생성하는 메서드 만들기
```dart
  // 1. Create : 데이터 쓰기 메서드
  Future<bool> insert({
    required String title,
    required String content,
    required String writer,
    required String imageUrl,
  }) async {
    // try catch 예외 처리
    try {
      // 1) 파이어스토어 인스턴스 가지고 오기
      final firestore = FirebaseFirestore.instance;
      // 2) 컬렉션 참조 만들기
      final collectionRef = firestore.collection('posts');
      // 3) 문서 참조 만들기
      // doc()에 문자열 타입에 파라미터를 넣으면 수동으로 ID 생성
      final docRef = collectionRef.doc();
      // 4) set()메서드로 값 쓰기
      // set은 만약 id를 수동으로 생성 시 같은 아이디가 존재하면 덮어쓰기
      docRef.set({
        'title': title,
        'content': content,
        'writer': writer,
        'imageUrl': imageUrl,
        'createdAt': DateTime.now().toIso8601String(),
      });
      return true;
    } catch (e) {
      print(e);
      return false;
    }
  }
```

**Read**

- 데이터를 서버에서 불러오는 메서드 만들기
```dart
  // 2. Read : 특정ID의 데이터(필드)를 불러오는 메서드
  Future<Post?> getOne(String id) async {
    try {
      // 1) 파이어베이스 파이어스토어 인스턴스 가지고 오기
      final firestore = FirebaseFirestore.instance;
      // 2) 컬렉션 참조 만들기
      final collectionRef = firestore.collection('posts');
      // 3) 문서 참조 만들기
      final docRef = collectionRef.doc(id);
      // 4) get()메서드로 데이터 가지고 오기
      final doc = await docRef.get();
      return Post.fromJson({
        'id': id,
        ...doc.data()!,
      });
    } catch (e) {
      print(e);
      return null;
    }
  }
```

**Update**

- 서버에 있는 데이터를 수정하는 메서드 만들기
```dart
  // 3. Update : 데이터를 수정하고 다시 서버에 업데이트
  Future<bool> update({
    required String id,
    required String title,
    required String writer,
    required String content,
    required String imageUrl,
  }) async {
    try {
      // 1) 파이어스토어 인스턴스 가지고 오기
      final firestore = FirebaseFirestore.instance;
      // 2) 컬렉션 참조 만들기
      final collectionRef = firestore.collection('posts');
      // 3) 문서 참조 만들기
      final docRef = collectionRef.doc(id);
      // 4) 값을 업데이트 해주기
      // 업데이트할 값 Map 형태로 넣어주기
      // .update()메서드는 id에 해당하는 문서가 없을 때 에러 발생
      await docRef.update({
        'title': title,
        'writer': writer,
        'content': content,
        'imageUrl': imageUrl,
      });
      return true;
    } catch (e) {
      print(e);
      return false;
    }
  }
```

**Delete**

- 서버에 있는 데이터를 삭제하는 메서드 만들기
```dart
// 4. Delete : 서버에 있는 특정 ID의 데이터 삭제
  Future<bool> delete(String id) async {
    try {
      // 1) 파이어스토어 인스턴스 가지고 오기
      final firestore = FirebaseFirestore.instance;
      // 2) 컬렉션 참조 만들기
      final collectionRef = firestore.collection('posts');
      // 3) 문서참조 만들기
      final docRef = collectionRef.doc(id);
      // 4) delete 메서드로 삭제
      await docRef.delete();
      return true;
    } catch (e) {
      print(e);
      return false;
    }
  }
```

### ViewModel (Riverpod)

- 서버나 API에서 받은 데이터를 상태 변화에 사용하기 위한 로직을 구성하는 곳
- riverpod 사용 시 flutter_riverpod패키지 설치 필수
**1. 상태 클래스 만들기**

- 상태 클래스 만들기
```dart
class HomeState {
  final User? user;      // 데이터를 사용할 Model 클래스
  final bool isLoading;  // 상태 관리 사용할 추가적인 데이터

  HomeState({this.user, this.isLoading = false});
}
```

- 만약 상태 관리를 딱 Model 클래스의 데이터만 관리한다면 상태 클래스를 생성할 필요 없음
**2. 뷰모델 만들기**

- Notifier 상속 받아 ViewModel 만들기 (기본 구조)
```dart
// Notifier를 상속 받아 만들고 제네릭 안에 관리해야될 타입을 넣음(상태 클래스)
class HomeViewModel extends Notifier<HomeState> {
 }
```

- 상태의 초기 값을 반환하여 초기 상태를 정의
```dart
class HomeViewModel extends Notifier<HomeState> {
  @override
  HomeState build() {
		// 초기값을 반환해줌
    return HomeState([]);
  }
 }
```

- state에 새로운 상태를 할당하여 상태를 변경하는 메서드 만들기 (비동기로 진행)
```dart
class HomeViewModel extends Notifier<HomeState> {
  @override
  HomeState build() {
    return HomeState([]);
  }

	// 상태가 병경되는 메서드 만들기
  Future<void> searchBooks(String query) async {
	  // 데이터 받을 객체를 vookRepository에 할당
    final bookRepository = BookRepository();
    // 객체의 데이터 요청 메서드 실행 후 books에 할당
    final books = await bookRepository.searchbooks(query);
    // 변경되는 데이터 state에 할당
    state = HomeState(books);
  }
}
```

**3. 뷰모델 관리자 만들기**

- Riverpod에서UI와 상태를 연결하려면 Provider의 등록이 필요함
- 뷰모델 Provider 관리자 만들기 
```dart
// Provider 관리자 선언, 제네릭에 상태를 관리할 타입, 상태관리될 타입 명시
final homeViewModelProvider =
    NotifierProvider<HomeViewModel, HomeState>(() {
      return HomeViewModel();
    });
```

**뷰모델에 상속되는 Notifier의 종류 (합쳐서 사용 가능)**

- Notifier : 상태 클래스에 있는 상태를 직접 관리하는 기본 클래스
**Async : 로딩, 결과, 에러를 표시하는 상태가 기본으로 내장되어 있는 클래스**

- 주로 로딩, 에러 상태를 표현해 주고 싶을 때 사용
- 뷰모델 (Notifier앞에 붙여서 사용)
```dart
class UserViewModel extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    // 최초 빌드 시 비동기 데이터 불러오기
    return await UserRepository().getUser();
  }

  Future<void> refreshUser() async {
    state = const AsyncValue.loading(); // 1. 로딩 상태
    try {
      final user = await UserRepository().getUser();
      state = AsyncValue.data(user); // 2. 성공 상태
    } catch (e, st) {
      state = AsyncValue.error(e, st); // 3. 실패 상태
    }
  }
}

// Provider 관리자 선언, 제네릭엔 상태를 관리할 타입과, 상태가 될 타입 명시
final userProvider = AsyncNotifierProvider<UserViewModel, User>(
  () => UserViewModel(),
);
```

- UI에서 상태 관리 
```dart
@override
Widget build(BuildContext context) {
  final userState = ref.watch(userProvider);

	// 상태 관리를 받은 userState에 로딩, 결과, 에러에 대한 UI 구현이 가능
  return userState.when(
    loading: () => const CircularProgressIndicator(),
    data: (user) => Text('이름: ${user.name}'),
    error: (e, st) {
      print(st); // 디버깅용
      return Text('에러 발생: $e');
    },
  );
}
```

**Family : build에 파라미터를 받을 수 있게 해주는 클래스**

- ViewModel 클래스에 파라미터를 받고 싶을 때 사용
- 뷰모델 (주로 Notifier 앞에 사용하며 Async가 있을 경우 AsyncFamilyNotifier로 사용)
```dart
// FamilyNotifier제네릭에는 관리하는 상태 타입과, 받을 파라미터 타입을 쓴다
class MyViewModel extends FamilyNotifier<MyState, String> {
  @override
  MyState build(String param) {
    // 파라미터 param을 사용해서 초기 상태 반환
  }
}

// Provider 관리자 선언, 제네릭엔 상태를 관리할 타입, 상태 관리될 타입, 파라미터 타입
final myProvider =
    NotifierProvider.family<MyViewModel, MyState, String>(
  MyViewModel.new,
);
```

- UI 
```dart
// 매개변수로 넣어주면 그 값이 파라미터로 넘어감
ref.watch(userProvider('123'));
```

**AutoDispose : 사용되지 않을 때 메모리에서 자동으로 해제되는 클래스**

- 주로 화면이 닫히거나, 위젯이 사라질 때 사용(메모리 누수 방지)
- 모든 Notifier 종류에서 관리자에 옵션으로 .autoDispose가 사용이 가능
  - 즉, 굳이 뷰모델 상속에 AutoDispose를 안 써도 됨 (가독성, 의도 표현 측면에서 사용)
- 뷰모델 (주로 Notifier 제일 앞에 사용) 
  - Provider 관리자에서 .autoDispose를 사용하면 UI에서는 신경 쓸 필요 없음
```dart
class CounterNotifier extends AutoDisposeNotifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
}

// Provider 관리자 선언, 제네릭엔 상태를 관리할 타입, 상태관리될 타입
final counterProvider = NotifierProvider.autoDispose<CounterNotifier, int>(
  () => CounterNotifier(),
);
```

### View (Riverpod)

**RiverPod의 전역 상태 관리 설정**

- 앱 내 모든 곳에서 상태 관리를 하기 위해 main 함수 최상단 루트에 ProviderScope 위젯으로 감싸기 
```dart
void main() {
  runApp(
    ProviderScope(   // ← 여기서 전체 앱을 감싸줌
      child: MyApp(),
    ),
  );
}
```

**UI 상태 관리 방법**

**ConsumerWidget**

- 상태를 직접 관리하지 않고 전체 위젯을 Provider 상태와 연동하여 상태 변화 때 전체를 리빌드
- StatelessWidget과 매우 유사
```dart
class MyPage extends ConsumerWidget {
  // build에 파라미터로 ref(riverpod 상태 관리 도구)를 받아야함
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 상태가 변경되는 데이터를 관리 도구를 사용하여 할당 (도구 : watch)
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Counter: $count')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // 상태가 변경되는 데이터를 관리 도구를 사용하여 할당 (도구 : read)
            ref.read(counterProvider.notifier).increment();
          },
          child: Text('증가'),
        ),
      ),
    );
  }
}
```

**ConsumerStatefulWidget**

- 상태를 직접 관리하고 Provider 상태와 연동이 가능하며 상태 변화 시 전체를 리빌드
- 직접 상태 관리는 주로 UI에서만 필요한 상태를 말함(즉, 보이는 화면에서만 변하는 값)
- StatefulWidget과 매우 유사 
```dart
class MyPage extends ConsumerStatefulWidget {
  @override
  ConsumerState<MyPage> createState() => _MyPageState();
}

class _MyPageState extends ConsumerState<MyPage> {
  // 자체 상태
  int localCount = 0;

  // ConsumerStatefulWidget는 ref를 가지고 있어 파라미터로 받지 않음
  @override
  Widget build(BuildContext context) {
    // 상태가 변경되는 데이터를 관리 도구를 사용하여 할당 (도구 : watch)
    final globalCount = ref.watch(counterProvider);

    return Column(
      children: [
        Text('Local: $localCount, Global: $globalCount'),
        ElevatedButton(
          // 자체 상태 변경
          onPressed: () => setState(() => localCount++),
          child: Text('Local +1'),
        ),
        ElevatedButton(
          onPressed: () =>
              // 상태가 변경되는 데이터를 관리 도구를 사용하여 할당 (도구 : read)
              ref.read(counterProvider.notifier).increment(),
          child: Text('Global +1'),
        ),
      ],
    );
  }
}
```

**Consumer**

- Stateless/Stateful 위젯에서 사용되며 전체를 리빌드하지 않고 상태 변화되는 부분만 리빌드
- 상태 변화가 발생하는 위젯을 Consumer로 감싸서 사용
```dart
class MyPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    print('전체 build 실행됨');
    return Column(
      children: [
        Text('고정된 텍스트'),
        // 상태 변화가 발생할 위젯을 Consumer로 감쌈
        Consumer(
          // builder에서 필요한 파라미터를 넣어줌 (context, ref, child)
          builder: (context, ref, child) {
            // 상태가 변경되는 데이터를 관리 도구를 사용하여 할당 (도구 : watch)
            final count = ref.watch(counterProvider);
            print('Consumer 부분만 리빌드');
            return Text('count: $count');
          },
        ),
      ],
    );
  }
}
```

**ProviderListener**

- UI를 리빌드 하지 않고 상태 변화에 따른 부수 효과를 나타내고 싶을 때 사용
- 상태 관리 도구에 listen이랑 비슷, ProviderListener는 부수 효과를 선언적으로 처리
**Riverpod 상태 관리 도구 종류 및 사용 방법**

- 상태 관리 도구 종류
**watch(Provider) : 상태가 변경되면 자동으로 위젯을 리빌드 (UI에서 상태 표시 등)**

```dart
  final count = ref.watch(counterProvider); // 구독
  return Text('$count'); // 상태 변경 시 자동 리빌드
```

**read(Provider) : 상태를 1회성으로 빌드, 상태가 변해도 리빌드 되지 않음 (이벤트 처리 등)**

```dart
ElevatedButton(
  onPressed: () {
    ref.read(counterProvider.notifier).increment(); // 1회성 읽기 후 상태 변경
  },
  child: Text('증가'),
);
```

**listen(Provider, listener) : 상태 변화를 감지하고 콜백 실행 (UI와 별개로 알림 등에 사용)**

```dart
ref.listen<int>(counterProvider, (previous, next) {
  if (next > 10) {
    print('10을 초과했습니다!');
  }
});
```

**invalidate(Provider) : 상태를 초기 상태로 강제 리셋 (캐시를 비우거나 새 데이터 불러올 때)**

```dart
ElevatedButton(
  onPressed: () => ref.invalidate(counterProvider),
  child: Text('리셋'),
);
```

**사용 방법**

- 기본 형태 : ref.도구(Provider);
- 뷰모델의 메서드를 사용할 경우 : ref.도구(Provider.notifier).메서드();