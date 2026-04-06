---
title: "API 및 Firebase서버 연동"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 API 및 서버 연동

## API 연동

**API 연동 및 데이터 요청 흐름**

**사용할 API 구조 알기**

- 사용할 API의 URL, ID, Secret 받기
**HTTP 라이브러리 사용**

- Flutter에 터미널에서 http 패키지 추가 
```dart
flutter pub add http
```

**헤더와 파라미터 설정**

- API의 헤더와 파라미터 설정을 Thunder Client 등으로 확인
- 파라미터는 쿼리 스트링 형식으로 전달 (?다음부터 파라미터, &로 파라미터 추가)
  - 파라미터 종류는 API의 종류에 따라 다름 (해당 API의 공식 문서 사용법을 참조)
  - 네이버 지역 검색 API의 파라미터
**repository model 클래스 생성**

- repository 모델 클래스를 만들어 데이터 요청하기
```dart
import 'package:http/http.dart';
import 'dart:convert';

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
      // 어떤 곳에서 요청했는지 알 수 있는 ID와 Secret 넣기
      headers: {
        'X-Naver-Client-Id': 'YOUR_CLIENT_ID',
        'X-Naver-Client-Secret': 'YOUR_CLIENT_SECRET',
      },
    );
```

- API에서 받은 데이터 변환 
```dart
// 변환한 데이터 Map 타입의 map에 저장
Map<String, dynamic> map = jsonDecode(response.body);
// JSON에서 필요한 'item'에 데이터가 리스트로 구성되어 리스트로 뽑아서 저장
final items = List.from(map['items']);
```

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

**응답 처리**

- HTTP 상태 코드를 통하여 응답 처리 
```dart
    // 통신 코드가 200이면 정상 통신
    if (response.statusCode == 200) {
      // 변환한 데이터 Map 타입의 map에 저장
      Map<String, dynamic> map = jsonDecode(response.body);
      // JSON에서 필요한 'item'에 데이터가 리스트로 구성되어 리스트로 뽑아서 저장
      final items = List.from(map['items']);
      // 리스트의 각 요소를 하나씩 꺼내 새로운 Iterable 타입으로 저장
      final iterable = items.map((e) {
        return Book.fromJson(e);
      });
      
      // Iterable 타입에 저장된 요소들을 리스트화
      final list = iterable.toList();
      return list;
    }
    
    // 요청 실패 시 빈 리스트 반환
    return [];
```

- HTTP 상태 코드에 따른 오류 종류
## Firebase 연동

**Firebase 연동 및 데이터 요청 흐름**

**사용할 Firebase 프로젝트 생성하기**

- 프로젝트 생성으로 사용할 데이터베이스를 생성하기
- 데이터베이스에서 사용할 데이터 컬렉션 만들기(데이터 관리할 폴더라고 생각하기)
  - 데이터베이스 데이터 정규화 구조 : 컬렉션으로 나눠서 관리
```dart
// 컬렉션 1
컬렉션 (Collection) = 데이터의 상위 폴더
  └ 문서 (Document) = 하나의 객체/문서 (고유 ID를 가짐: 문서 명)
      └ 필드 (Field) = key-value 쌍으로 실제 데이터 (문서에 들어있는 값)

// 컬렉션 2
컬렉션 (Collection) = 데이터의 상위 폴더
  └ 문서 (Document) = 하나의 객체/문서 (고유 ID를 가짐: 문서 명)
      └ 필드 (Field) = key-value 쌍으로 실제 데이터 (문서에 들어있는 값)
```

  - 데이터베이스의데이터 역정규화 구조 : 하나의 컬렉션 안에 모든 데이터 관리
```dart
컬렉션 (Collection) = 최상위 폴더
  └ 문서 (Document) = 문서
      └ 컬렉션 (Collection) = 문서에 분기를 나누는 폴더
          └ 문서 (Document) = 분기별로 나눠진 폴더에 있는 문서
              └ 필드 (Field) = 해당 문서에 존재하는 데이터
```

**Firebase CLI로 프로젝트 연동**

- 1 ~ 3까지는 한번만 진행하면 다음 프로젝트에서도 사용 가능
1. Firebase CLI 설치 (Window의 경우 Node.js 먼저 설치 필요)
1. Firebase CLI를 통해 Firebase 계정에 로그
```dart
firebase login
```

1. 터미널에 입력하여 FlutterFire CLI 설치 (에러 시 환경 변수 path 등록 확인 후 수동으로 등록)
```dart
dart pub global activate flutterfire_cli
```

1. 프로젝트 연동(CLI 초기화 및 프로젝트를 Firebase와 연결과 필요한 파일 자동 다운) 
```dart
// vsCode 터미널에서 입력
flutterfire configure
```

  - configure 이후 Firebase 프로젝트 선택 → 플랫폼(Android/IOS) 선택 → 자동 설정
1. Firebase core 패키지 추가 (Firebase SDK 초기화 담당)
```dart
// vsCode 터미널에서 입력
flutter pub add firebase_core
```

1. cloud_firestore 패키지 추가 (DB를 사용할 수 있도록 담당)
```dart
// vsCode 터미널에서 입력
flutter pub add cloud_firestore
```

1. 프로젝트 main에서 Firebase 서비스를 사용하기 위한 초기화 작업 진행  (중요)
```dart
void main() async {
  // flutter 프레임워크(다트 코드)와 네이티브 엔진(Android/iOS 코드) 연결을 보장
  // main에서 비동기 작업 시 꼭 호출해야 문제가 안생김
  WidgetsFlutterBinding.ensureInitialized();

  // 서버와 앱을 연결하고 초기화 하는 작업
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  runApp(ProviderScope(child: MyApp()));
}
```

**Firebase 서버 요청 흐름**

- repository 클래스로 서버(Firebase)에 데이터 요청하기
```dart
class PostRepository {
  Future<List<Post>?> getAll() async {
      // 1. 파이어스토어 인스턴스 가지고 오기
      final firestore = FirebaseFirestore.instance;
      // 2. 컬렉션 참조 만들기
      final collectionRef = firestore.collection('posts');
      // 3. 값 불러오기
      final result = await collectionRef.get();
```

- 파싱 (Json → Map 변환)
  - Firebase의 경우 데이터를 Map타입으로 받아서 변환이 필요 없음
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

**Firebase 서버 데이터 관리**

- 서버에 사용되는 데이터의 관리는 repository 클래스에서 C.R.U.D  메서드로 관리
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

**예외 처리**

- C.R.U.D 메서드는 try catch로 예외 처리하여 에러 방지