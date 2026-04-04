---
title: "setState"
status: "published"
date: "2026-04-04"
type: "posts"
---

# setState

# 📚 setState 상태 관리 한계

## StatefulWidget의 setState 상태 관리 및 한계

### StatefulWidget의 setState 상태 관리

- (TIL : 2025-05-30 위젯의 구성 → StatfulWidget 참고)
- setState의 상태 관리는 동적 변화를 주기 위해 필요하다. 
- setState가 없으면 데이터의 상태는 변화하지만 위젯의 UI를 데이터 상태의 변화에 맞춰 다시 그리는 
함수가 없어 동적 변화를 하지 않는다.
### setState의 한계

- 부모위젯 안에 여러 자식위젯의 상태를 공유하거나 전달할 때 구조를 만들기 어렵고 복잡하며 규모가 커질수록 모든 상태 변경을 중앙에서 관리하지 않기 때문에 더욱 코드가 산만해지고 유지 보수가 어렵다.
- setState는 StatefulWidget전체를 다시 빌드하기 때문에 작은 상태 변경에도 전체 위젯들을 다시 그려 효율이 떨어지고 성능이 저하될 수 있다.
- setState는 StatefulWidget의 내부에서만 상태를 변경한다. 즉, 위젯 트리의 특정 부분에서만 상태를 변경하기 때문에 앱 전체의 상태를 관리하거나, 전체에서 상태를 접근, 변경, 감지하기 어렵다.
- setState 구조로 구성된 위젯 트리의 이벤트 및 상태 변화
  - GestureDetector (left) 와 Text가 서로 관계없고 연결되어 있지 않기 때문에 
상태를 전달할 수 없다.
  - 상태 및 이벤트 등은 부모를 통해 여러 차례 거쳐서 전달해 줘야 한다.
  - 이것은 복잡하고 번거로운 작업이며, 부모의 상태 변경에 따라 불필요한 위젯들 까지 모두 다시 그려야 해서 성능적인 이슈가 발생한다.
![](/images/07_setState/img_01.png)

### 중앙 집중적인 상태 관리

- 이러한 setState의 한계를 해결하기 위하여 중앙 집중적인 상태 관리 라이브러들이 개발되었다.
  - 중앙 집중 상태 구조로 구성된 위젯 트리의 이벤트 및 상태 변화
    - 중앙 집중 상태 관리를 통해 어디서든 상태를 관리, 이벤트를 주고 받을 수 있다.
    - 불필요한 전달 과정이 없고, 필요한 영역만 화면을 갱신하기 때문에 성능적으로 
매우 우수하다.
![](/images/07_setState/img_02.png)

- 대표적인 중앙 집중 상태 관리 라이브러리 
  - Provider         : Flutter 공식 패키지, 간단하고 직관적
  - Riverpod        : Proivider의 발전형, 더 안전하고 유연
  - Bloc/Cubit      : 구조가 명확하고 규모가 있는 앱에 강함
  - Getx               : 코드가 간결하고 빠름, 학습 곡선이 낮음
  - Flutter 라이브러리 분류 : 공식 vs 서드파트(3rd-party)
  ⚠️ Flutter 라이브러리 사용 방법

  - Flutter 에서 1st-party든 3rd-party든 SDK 기본 제공이 아닌 기능을 쓰기 
위해선 반드시 다음 두 단계를 거쳐야 함
    1️⃣ pubspec.yaml에 추가

    - 사용할 패키지의 원하는 버전을 수동으로 입력 (버전 고정용)
```yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.0  // 원하고자 하는 버전 명시
```

    - 그 다음에 터미널에서 패키지 설치를 실행
```bash
flutter pub get
```

    - 또는, 최신 버전 패키지를 터미널을 통해 자동으로 설치 방법
```bash
flutter pub add provider
```

    2️⃣ 코드에서 import로 불러오기

    - 코드 맨 위에 해당 패키지를 import로 불러오기
```dart
import 'package:provider/provider.dart';
```

# 🤔 회고

## 현재 학습 계획 정리

- 오늘은 짧게 공부를 마치고 학습의 방향과 흐름에 대해서 점검을 하는 시간을 가졌다.
- 지금까지 배운 학습의 흐름이다.
  1. 문법의 기초  : 변수, 함수, 동기/비동기, 클래스, 위젯트리
  1. 위젯의 구성 : Stateless/Stateful, Life Cycle
  1. 위젯의 종류 : View위젯, Layout 위젯, 기능성 위젯
  1. 상태관리 (진행중) : setState의 한계
  - 여기서 상태 관리는 setState, Provider, Riverpod, Bloc/Cubit, GetX 가 있는데 강의에서는 GetX가 가장 인기 있고 사용이 매우 쉽다고 하여 알려주고 있다.
  - 상태 관리의 흐름으로 볼 땐  setState → Provider → Riverpod → Bloc/Cubit → GetX 순으로 공부하는 것이 좋아 보여 학습 계획에 고민이 생긴다.
  - 고민의 끝으로 기초부터 천천히 다지는게 좋을 거 같다는 생각이 들어 흐름대로 공부 계획을 세워야 겠다.
## 앞으로의 학습 계획 수립

- 전체적인 학습 계획을 다시 한번 점검해 보는게 좋을거 같아서 학습 흐름을 살펴보고 계획을 수립했다.
  1. 문법의 기초
  1. 위젯의 구성 및 생명주기
  1. 위젯의 종류
  1. 상태 관리
  1. Firebase
  1. API 통신
  1. 구글 애드몹 연동
  1. 배포
- 추가적인 시스템 사용
  - Git : 버전 관리 시스템, 코드 변경 내용을 기록해서 이전 상태로 돌아가거나 함께 작업이 가능해 
       앱 개발할 때 필 수 도구 중 하나 (별도의 설치 필요)
  - Flutter DevTools : 디버깅, 성능분석, UI 검사 등을 할 수 있게 해주는 도구 모음 시스템, 
                             앱이 커지고 복잡해 질 때 유용하게 사용 (Flutter 제공)
- Flutter 학습 이후 추가적으로 학습해야 할 것
  - 네이티브 연동 (Platform Channel) : Android(Java/Kotlin), iOS(Swift/Obj-C) 쪽 코드와 Flutter 
                                                       코드를 연결 해주는 고급 주제
  - 접근성 : 앱 사용 편의성과 관련된 설정이나 OS 레벨 지원 기능을 다루는 영역
  - 국제화 : Flutter 앱을 여러 언어와 지역에 맞게 표시할 수 있도록 준비하는 과정
# 📒 New 용어 정리
