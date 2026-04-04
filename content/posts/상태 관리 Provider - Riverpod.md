---
status: "published"
date: "2026-04-04"
type: "posts"
---

# 상태 관리 Provider / Riverpod

# 📚 집중 상태 관리

## Provider 기반 상태 관리 : 공식 vs 서드파티

### Provider 상태 관리                               

- Provider는 Flutter 앱에서 상태(데이터)를 공유하고 관리하는 클래스이다.
  - 즉, 위젯 트리 어딘가에 Provider<MyData>를 심어서 자식 위젯들이 context를 통해 MyData를 꺼내 쓸 수 있게 해준다.
- Provider의 필수 구성 요소는 Provider<> + create() + child 이다.
```dart
// 예시
Provider<MyData>(                      // <> 안은 공유하고 싶은 데이터 타입
  create: (context) => MyData(),       // create는 데이터를 만드는 함수
  child: MyApp(),                      // MyData를 사용하는 모든 위젯
);
```

**Provider 사용 전 세팅**

1. Provider 패키지 설치하기
**수동으로 Provider 패키지 설치하기**

- pubspec.yaml에 원하는 Provider 버전을 추가하기
```dart
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.1   # (버전은 최신으로!)
```

- 터미널에서 명령어를 실행하여 해당버전 패키지 설치하기
```bash
flutter pub get
```

**자동으로 Provider 패키지 설치하기**

- 터미널에서 명령어를 실행하면 자동으로 yaml에 최신 버전이 추가되며 설치된다.
```bash
flutter pub add provider
```

- 또는 터미널을 통해 해당 버전을 자동으로 추가하여 설치도 가능하다.
```bash
flutter pub add provider --version ^6.0.0
```

1. Dart 파일 상단에 import 하기
- Provider에서 사용하는 파일을 쓰기 위해선 반드시 import를 해야한다.
```bash
import 'package:provider/provider.dart';
```

1. Provider를 runApp()에 감싸기
  - Provider는 앱의 상태를 공급하기 때문에 보통 main.dart에서 runApp()에 감싸준다.
```dart
void main() {
  runApp(
    Provider<MyData>(
      create: (context) => MyData(),
      child: MyApp(),
    ),
  );
}
```

- Provider에서 자주 사용되는 옵션
- Provider를 쓸 때 UI를 빌드하는데 자주 사용되는 도우미 메서드들
- Provider의 주요 단점
  - context에 의존함 : 상태를 읽으려면 항상 BuildContext가 필요
  - 테스트의 어려움 : 위젯에 의존적이라 테스트 코드 작성이 불편
  - 전역 상태 관리의 불편 : 위젯 트리에 종속되어 있어 전역 상태 관리에 제약이 있음
  - 자동 정리 어려움 : Provider를 안 쓰게 될 때 메모리 정리가 어려울 수 있음
- 이러한 단점들을 보완하여 Provider의 상위 버전이 Riverpod 이다.
### Riverpod 상태 관리                     

- Riverpod는 기존 Provider와 다르게 상태와 UI를 완전히 분리해서 관리한다.
- Riverpod 사용 준비 세팅하기
  - pubspec.yaml에 패키지 추가 및 설치 하기
```bash
flutter pub add flutter_riverpod
```

  - 필요한 곳에 import명령어 사용하기
```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
```

  - main.dart에 ProviderScope 추가하기
    - Riverpod는 모든 Provider를 ProviderScope라는 위젯으로 감싸야 작동한다.
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    ProviderScope(  // 여기에 ProviderScope로 앱 전체 감싸기
      child: MyApp(),
    ),
  );
}
```

- Riverpod의 필수 구성 요소
  1. Provider 선언
    - final+counterProvider(변수) =Provider<T>((ref) => 0))(상태관리 객체의 타입)
```dart
final counterProvider = StateProvider<int>((ref) => 0);
```

    - Riverpod의 주요 Provider 타입
    - Provider 타입에 따른 상태 변화 방법
      1. StateProvider타입은 .notifier로 직접 상태 값을 바꾼다.
      1. StateNotifierProvider나 ChangeNotifierProvider같은 notifier가 들어간 타입은 내부에서 메서드를 사용하여 상태를 변경하고 관리한다.
      1. FutureProvider나 StreamProvider는 비동기나 실시간 데이터 흐름을 다루기 때문에 상태를 직접 컨트롤하는 게 아니라, 외부에서 데이터 변화를 구독해 상태가 바뀌는 형태이다.
    1. UI에서 사용
    - ConsumerWidget 또는 Consumer를 사용해 상태를 사용하는 UI를 만들고 그 안에서 WidgetRef 타입의 ref를 이용해 상태를 watch하거나 read하는 등 구독한다.
    - 상태 변화는 ‘Provider 타입에 따른 상태 변화 방법’에 따라 상태를 변화한다.
      - 여기에선 StateProvider를 사용하였기 때문에 counterProvider.notifier를 사용한다.
```dart
class MyHomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider); // 상태 구독
    // 즉, counterProvider 상태 변화를 ref.watch가 감지해서, 
    // 최신 상태 값을 count 변수에 담아라
     
    return Scaffold(
      body: Center(child: Text('Count: $count')),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(counterProvider.notifier).state++,
      ),
    );
  }
}
```

- Riverpod 주요 도우미 함수들
  - ref.watch() : 상태를 구독해서 값이 바뀌면 위젯을 다시 그림
```dart
// 주로 UI 빌드 메서드 안에서 사용
final count = ref.watch(counterProvider);
```

  - ref. read() : 상태를 호출될 때 한 번만 읽음
```dart
// (onPressed 등) 주로 이벤트 핸들러에서 사용
ref.read(counterProvider.notifier).state++;
```

  - ref.listen() : 상태 변화를 감지해서 특정 동작을 실행 (UI를 다시 그리지 않음)
```dart
//보통 initState나 효과 로직에 사용
ref.listen(counterProvider, (prev, next) {
  print('값이 $prev → $next 로 바뀜');
});
```

  - ref.invalidate() : 특정 Provider의 상태를 강제로 다시 계산
```dart
// 주로 캐시 리센, 새로고침 등에 사용
ref.invalidate(counterProvider);
```

# 🔧 실습 & 개발

- 아침에 코딩 문제
```dart
* 문제
input : 정수 배열 nums와 정수 target이 주어질 때, 

두 수를 더해서 / target이 되는 

배열 내 두 숫자의 인덱스를 반환하세요.

같은 요소를 두 번 사용할 수 없습니다.
답은 반드시 하나만 존재한다고 가정합니다.

Follow-up: Can you come up with an algorithm that 
is less than O(n^2) time complexity?

* 조건
2 <= nums.length <= 10^4
-10^9 <= nums[i] <= 10^9
-10^9 <= target <= 10^9
정답은 하나만 존재하며, 중복된 입력이 없습니다.

* 예시
예제 1
	입력: nums = [2, 7, 11, 15], target = 9
	출력: [0, 1]
	설명: nums[0] + nums[1] = 2 + 7 = 9 이므로 [0, 1] 반환

예제 2:
	입력: nums = [3, 2, 4], target = 6
	출력: [1, 2]
	설명: nums[1] + nums[2] = 2 + 4 = 6 이므로 [1, 2] 반환
	
예제 3:
	입력: nums = [3, 3], target = 6
	출력: [0, 1]
	설명: nums[0] + nums[1] = 3 + 3 = 6 이므로 [0, 1] 반환
```

- 문제 풀이
  - 처음 접하는 실습 문제 접근 방법에 대해 아무것도 모르겠다.
```dart
class Solution {
  List<int> twoSum(List<int> nums, int target) {
   add (int a, int b) {
    a + b = target
    };
   return target;
  };
}
```

  - 해답 풀이
```dart
List<int> twoSum(List<int> nums, int target) {
  // Step 1: 숫자와 인덱스를 저장할 해시맵 생성
  Map<int, int> map = {};

  // Step 2: 배열을 순회하면서 필요한 값을 맵에서 찾음
  for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i]; // 필요한 값 계산

    // Step 3: 필요한 값이 맵에 존재하면 정답 반환
    if (map.containsKey(complement)) {
      return [map[complement]!, i];
    }

    // Step 4: 현재 숫자와 인덱스를 맵에 추가
    map[nums[i]] = i;
  }

  // Step 5: 문제 조건에 의해 정답이 항상 존재하므로 여기에는 도달하지 않음
  throw ArgumentError("No solution found");
}
```

  - 해석
```dart
class Solution {
  List<int> twoSum(List<int> nums, int target) {
  // int 타입의 List를 생성하고 twoSum이라는 함수 사용 
  // twoSum 함수의 매개변수로 List<int> nums, int target을 사용
  
  Map<int, int> map = {};
  // twoSum 함수의 코드는 Map이라는 key와 value를 저장하는 클래스를 사용해
  // Map<int, int>는 key와 value 모두 int 타입임을 의미해
  // 이 map이라는 변수는 빈 map 객체를 만들어서 초기화된 상태

  for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i]; // 필요한 값 계산
  // int 타입 i는 0 이고, i가 nums의 길이 수보다 작을때만 i를 1씩 늘리며 반복해
  // int 타입 complement는 target 값에서 nums의 i 번째 위치의 값을 뺀 값

    if (map.containsKey(complement)) {
      return [map[complement]!, i];
    }
   // 만약 complement의 값이 map의 키에 있으면 
   // map에서 complement에 해당되는 값을 첫 번째 인덱스로
   // 현재 인덱스 i를 두번째 인덱스로 리스트를 반환해라

    map[nums[i]] = i;
   // if에 해당하지 않으면 즉, complement가 map에 없으면
   // nums[i]의 값을 key로 그 인덱스 i를 value로 map에 저장해
  }
  // Step 5: 문제 조건에 의해 정답이 항상 존재하므로 여기에는 도달하지 않음
  throw ArgumentError("No solution found");
  // 이 코드에 도달하면 함수 실행을 멈추고 No solution found라는 메세지와 함께 던져라
 }
}
```

  - 연속 for문으로 해보기
```dart
class Solution {
  List<int> twoSum(List<int> nums, int target) {
  for (int i = 0; i < nums.length; i++) {
   int first = i;
   for (int j = 0; i < nums.length; j++) {
    int j = target - first;
    return List [i,j];
  };
 }
```

  - 연속 for 문 해답
```dart
class Solution {
  List<int> twoSum(List<int> nums, int target) {
    for (int i = 0; i < nums.length; i++) {
      for (int j = i + 1; j < nums.length; j++) {
        if (nums[i] + nums[j] == target) {
          return [i, j];
        }
      }
    }
    throw ArgumentError("No solution found");
  }
}
```

  - 내가 만든 코드의 문제
    - 바깥쪽 루프: 첫 번째 숫자의 인덱스
    - 안쪽 루프: 두 번째 숫자의 인덱스 → 같은 숫자를 두번 사용할 수 없기 때문에 i+1부터 시작
    - 두 숫자의 합이 target이면 그때 [i,j] 반환 → if 문 사용
# 📒 New 용어 정리
