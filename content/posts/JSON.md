---
title: "JSON"
status: "published"
date: "2026-04-04"
type: "posts"
---

# JSON

# 📚 JSON

## JSON이란?

- JSON(JavaScript Object Notation) : JavaScript의 객체 표기 방식 기반의 문자열 형식 데이터 구조
**JS Object : 괄호 안에 여러 개의 key - value이 구성 **

```dart
[
	{
		"name": "이지원",
		"age": 20
	},
	{
		"name": "오상구",
		"age": 7,
		"isMale" : true,
		"favorite_foods" : ["삼겹살", "연어", "고구마"],
		"contact": {
			"mobile": "010-0000-0000",
			"email": null
		}
	}
]
```

  - key는 반드시 문자열
**value는 정해진 타입만 가능**

  String(문자열), Number(숫자), Boolean(논리), Array(대괄호), Object(중괄호), null

- 서버나 API에서 데이터를 주고 받을 때 JSON 형식으로 진행
**서버에서 JSON 데이터 받을 때 처리**

```dart
[JSON 문자열]
  ↓ jsonDecode    // JSON 문자열 -> Map<String, dynamic>
[Map<String, dynamic>]
  ↓ fromJson()    // 클래스내 네임드 생성자(fromJson)으로 Map -> Dart 클래스 객체
[내 Dart 클래스 객체]
```

  - jsonDecode() : 파싱 ⇒ JSON 문자열를 Dart 언어(Map<String, dynamic>)로 변환 
```dart
  String easyJson = """
{
	"name": "오상구",
	"age": 7,
	"isMale" : true
}
""";

  Map<String, dynamic> easyMap = jsonDecode(easyJson); // 파싱
```

  - 객체를 만들기 위해 class를 정의하고 네임드 생성자를 만들어 데이터를 받아 필드를 초기화
```dart
class Pet {
  String name;
  int age;
  bool isMale;

  Pet({
    required this.name,
    required this.age,
    required this.isMale,
  });

  // fromJson named 생성자 만들기
  Pet.fromJson(Map<String, dynamic> map)
      : this(
          name: map['name'],
          age: map['age'],
          isMale: map['isMale'],
        );
 }
```

  - fromJson() ⇒ Map<String, dynamic>을 Dart 클래스 객체로 생성 
```dart
  PetDetail petDetail = PetDetail.fromJson(easyJson);
```

**서버로 JSON 데이터 보낼 때 처리**

```dart
[내 Dart 클래스 객체]
  ↓ toJson()     // 클래스내 메서드를 통해 Dart 클래스 객체 -> Map
[Map<String, dynamic>]
  ↓ jsonEncode   // Map<String, dynamic> -> JSON 문자열
[JSON 문자열]
```

  - 클래스 객체를 Map타입으로 변환 해주기 위한 메서드 생성 
```dart
class Pet {
  String name;
  int age;
  bool isMale;

  Pet({
    required this.name,
    required this.age,
    required this.isMale,
  });

  // Map<String, dynamic> toJson 메서드 만들기
  Map<String, dynamic> toJson() {
    return {
      'name': name,
      'age': age,
      'isMale': isMale,
    };
  }
 }
```

  - toJson() ⇒ Dart 클래스 객체을 Map<String, dynamic>로 변환 
```dart
Map<String, dynamic> petToJson = pet.toJson();
```

  - JsonEncode() : 직렬화 ⇒ Dart 언어(Map<String, dynamic>)를 JSON 문자열로 변환 
```dart
String dataToServer = JsonEncode(petToJson);
```

**여러 개의 key-value가 있는 JSON 처리 방법**

  - JSON 코드 
```dart
List<String> jsonList
[                           // 리스트 타입으로 여러개 있는 경우
  {
    "name": "이지원",
    "age": 20
  },
  {
    "name": "오상구",
    "age": 7,
    "isMale": true,
    "favorite_foods": ["삼겹살", "연어", "고구마"],  // value가 리스트 인 경우
    "contact": {                     // value안에 또 다른 key-value 있는 경우
      "mobile": "010-0000-0000",
      "email": null
    }
  }
]
```

**JSON 문자열를 Dart 언어로 파싱**

  - List형태의 데이터를 파싱 
```dart
List<dynamic> parsedList = jsonDecode(jsonString);
```

  - 파싱 된 List를 Map<String, dynamic>타입으로 분리 
```dart
Map<String, dynamic> parsed0 = ParsedList[0];  // 리스트 첫 번째
Map<String, dynamic> parsed1 = ParsedList[1];  // 리스트 두 번째
```

**객체를 만들기 위한 클래스 정의 및 필드 초기화**

  - value안에 또 다른 Map이 있는 경우 해당 Map에 대한 클래스 생성
```dart
class Contact {
  String mobile;  // cotact에 이 값은 항상 있어야 해서 non-nullable라고 판단
  String? email;  // 이 값은 있어도 되고 없어도 되고 해서 nullable

  Contact({
    required this.mobile,
    this.email,
  });

  // fromJson 만들기
  Contact.fromJson(Map<String, dynamic> json)
      : mobile = json['mobile'],
        email = json['email'];

  Map<String, dynamic> toJson() {
    return {
      'mobile': mobile,
      'email': email,
    };
  }
}

```

  - 다음 전체에 대한 데이터를 Map으로 변환 
```dart
class Pet {
  String name;       // 필수 데이터 = non-nullable
  int age;           // 필수 데이터 = non-nullable
  bool? isMale;      // 선택 데이터 = nullable
  List<String>? favoriteFoods;  // 선택 데이터 = nullable
  Contact? contact;  // 선택 데이터 = nullable 
										 // 데이터가 있으면 Contact 클래스 속성 nullable를 따짐
  // 일반 생성자
  Pet({
    required this.name,
    required this.age,
    this.isMale,
    this.favoriteFoods,
    this.contact,
  });

  // 네임드 생성자 fromJson
  Pet.fromJson(Map<String, dynamic> json)
      : name = json['name'],
        age = json['age'],
        isMale = json['isMale'],
        favoriteFoods = json['favorite_foods'] != null
            ? List<String>.from(json['favorite_foods']) 
            : null,   // 새로운 리스트로 타입 변환 dynamic -> String
        contact = json['contact'] != null
            ? Contact.fromJson(json['contact']) // contact 값 Map으로 호출
            : null;

  Map<String, dynamic> toJson() {
    return {
      'name': name,
      'age': age,
      'isMale': isMale,
      'favorite_foods': favoriteFoods,
      'contact': contact?.toJson(),
    };
  }
}
```

**객체 만들기**

  - 각각에 데이터에 대한 객체를 만듬
```dart
Pet pet0 = Pet.fromJson(parsed0); // 클래스에 같은 속성이 있으면 재사용 가능
Pet pet1 = Pet.fromJson(parsed1);
```

### factory 생성자

**factory : 내부에 코드를 쓸 수 있는 네임드 생성자를 만드는 키워드**

- 일반 생성자는 :으로  필드를 직접 초기화만 하고 그 구조 안에서 로직을 쓸 수 없음
- factory키워드를 사용하면 조건문, 예외 처리 등의 로직을 구현하고 return으로 객체 반환
```dart
class Pet {
  String name;
  int age;

  Pet({required this.name, required this.age});

  // factory 키워드가 붙은 네임드 생성자
  factory Pet.fromJson(Map<String, dynamic> map) {     // 로직 가능
    if (map['name'] == null) {
      throw Exception('이름은 반드시 필요합니다!');
    }

    return Pet(
      name: map['name'],
      age: map['age'] ?? 0, // 기본값
    );
  }
}

```

- 일반 생성자 vs 네임드 생성자 vs factory 네임드 생성자
# 📝 치트시트

## 📕 New 위젯 정리 :

## 📒 New 용어 정리
