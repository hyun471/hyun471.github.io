---
status: "published"
type: "posts"
title: "IBA를 실제 앱 개발에 적용했더니, 구조가 의도대로 나오지 않았다"
date: "2026-05-28"
description: "Island Bridge Architecture를 실제 앱 개발에 적용하며 발견한 문제와, 파일 단위 섬·계약 브릿지·문서 구조로 개편한 의사결정"
tags: ["Architecture", "IBA", "Flutter", "Refactoring", "Documentation"]
---

# IBA를 실제 앱 개발에 적용했더니, 구조가 의도대로 나오지 않았다

전에 `Island Bridge Architecture`, 줄여서 IBA라는 구조를 만들었다.

목표는 분명했다. AI 에이전트가 코드를 작성할 때 토큰을 덜 쓰고, 수정 범위를 좁히고, 서로 다른 역할의 파일을 함부로 건드리지 못하게 하는 것.

- 파일은 섬처럼 독립적이어야 한다.
- 섬끼리는 직접 연결하지 않는다.
- 연결은 `bridge/`를 통해서만 한다.
- AI는 bridge를 보고 영향 범위를 판단한다.

문제는 실제 프로젝트에 적용했을 때였다.

실제 앱 프로젝트를 만들면서 IBA 구조를 적용해봤는데, 결과물이 내가 의도한 IBA와 달랐다. 폴더는 생겼다. bridge도 생겼다. 그런데 구조는 제대로 작동하지 않았다.

겉모습은 IBA였지만, 실제로는 IBA가 아니었다.

## 실제로 생긴 문제

Flutter 앱의 `lib/` 구조는 대략 이렇게 생성됐다.

```text
lib/
  bridge/
    feature-model/
    feature-view/
    feature-viewmodel/
    viewmodel-model/
    viewmodel-view/
  feature/
    home/
  model/
  style/
  view/
  viewmodel/
```

처음 보면 그럴듯하다.

`feature`, `view`, `viewmodel`, `model`, `style`, `bridge`가 다 있다. 기존 IBA 소개글에서 말했던 구조와 비슷해 보인다.

그런데 파일을 열어보면 문제가 바로 보였다.

## 문제 1: view 디자인 패턴이 없어서 화면이 커졌다

홈 화면 파일이 거의 모든 UI를 품고 있었다.

```text
lib/view/home_screen.dart
```

이 파일은 1000줄이 넘었다.

홈 화면 안에 앱바, 날짜 행, 달성률, 카드 그리드, 빈 상태, 바텀 내비게이션, FAB, 팝업 오버레이까지 전부 들어갔다.

이건 내가 원했던 "파일 하나 = 섬 하나"가 아니었다.

파일은 하나지만, 그 안에 여러 섬이 섞여 있었다.

원인은 IBA 자체보다 view 쪽 디자인 패턴이 없었다는 데 있었다.

`view/`라는 나라만 정의했지, 화면을 조립하는 파일과 실제 UI 조각을 나누는 규칙을 명시하지 않았다. 그러니 AI 입장에서는 `home_screen.dart` 하나에 계속 UI를 추가하는 것이 자연스러운 선택이었다.

내가 원했던 구조는 이런 쪽이었다.

```text
view/
  screen/
    home_screen.dart
  component/
    home/
      home_app_bar.dart
      home_date_row.dart
      home_achievement_row.dart
      home_task_card.dart
      home_bottom_nav.dart
      home_fab.dart
      home_expanded_popup.dart
```

`home_screen.dart`는 화면 조립만 해야 한다.

실제 UI 조각은 component 섬으로 나뉘어야 한다.

그런데 기존 지침은 `view/`를 하나의 큰 영역으로만 말하고 있었다. AI 입장에서는 "홈 화면 구현"이라는 요청을 받으면 `view/home_screen.dart` 하나에 계속 추가해도 된다고 해석할 수 있었다.

그래서 이 문제의 의사결정은 명확했다.

```text
view는 Screen Composer + Component Island 패턴을 사용한다.
```

이 결정은 브릿지 문제만큼 핵심은 아니지만, IBA가 실제 앱 UI에서 작동하려면 반드시 필요했다.

## 문제 2: bridge가 그냥 export 파일이 됐다

가장 큰 문제는 bridge였다.

내 의도에서 bridge는 계약이었다.

서로 다른 나라가 직접 알지 않도록, 주고받을 데이터와 함수 모양만 정의하는 경계여야 했다.

그런데 실제 생성된 bridge는 대부분 이런 식이었다.

```dart
// feature ↔ view 브릿지
export 'package:app_project/feature/home/task_completion.dart';
```

또는 이런 식이었다.

```dart
// viewmodel ↔ view 브릿지
export '../../viewmodel/home_viewmodel.dart' show homeProvider, HomeState, HomeNotifier;
export '../../bridge/viewmodel-model/home_model_bridge.dart'
    show MockSubTask, MockTask, MockPage, kMockPages;
```

이건 이번 개편의 핵심 문제였다.

이건 bridge가 아니다.

그냥 re-export다.

view가 feature를 직접 import하지 않았을 뿐, bridge가 feature를 그대로 노출하고 있다. 즉 "서로 모르게 한다"는 의도가 깨졌다.

겉으로는 bridge를 경유하지만, 실제로는 view가 feature의 함수와 viewmodel의 provider를 그대로 알고 있다.

이 구조에서는 feature의 이름이나 함수 모양이 바뀌면 view도 영향을 받는다. bridge가 완충 역할을 하지 못한다.

즉, 기존 IBA에서 가장 먼저 수정해야 할 의사결정은 이것이었다.

```text
bridge는 export 파일이 아니다.
bridge는 contract다.
```

이 결정을 하지 않으면 나머지 규칙은 의미가 약해진다. 아무리 폴더를 잘 나눠도 bridge가 내부 구현을 그대로 노출하면, 결국 직접 import를 다른 이름으로 부르는 것과 다르지 않다.

## 문제 3: bridge 종류가 너무 많고 기준이 흐렸다

두 번째로 큰 문제는 bridge의 단위였다.

기존 구조에는 이런 bridge 폴더가 생겼다.

```text
bridge/
  feature-model/
  feature-view/
  feature-viewmodel/
  viewmodel-model/
  viewmodel-view/
```

처음에는 나라끼리 모두 bridge로 연결하면 안전할 거라고 생각했다.

하지만 실제로는 판단 기준이 흐려졌다.

- model도 bridge를 타야 하나?
- style도 bridge를 타야 하나?
- view가 feature를 꼭 bridge로 볼 수 있나?
- bridge는 계약인가, export인가?
- feature-view bridge와 viewmodel-view bridge는 무엇이 다른가?

규칙이 많아지면 AI가 잘 따를 것 같지만, 실제로는 반대였다.

AI가 매번 "이번 연결은 어떤 bridge지?"를 판단해야 했다. 그러면 결과가 흔들린다.

여기서 내린 의사결정은 bridge 수를 무조건 줄이는 것이 아니라, bridge 생성 단위를 체계화하는 것이었다.

```text
bridge는 섬 단위가 아니라 feature area + country boundary 단위로 만든다.
```

이 결정으로 bridge 수는 상황에 따라 늘어날 수 있다. 대신 어떤 bridge를 왜 만드는지 기준이 생기고, 각 bridge의 책임을 명확하게 만들 수 있었다.

## 문제 4: map 파일이 있어도 충분하지 않았다

IBA에는 `app-map.md` 같은 map 파일을 두었다.

AI가 작업 전에 map을 읽고, 어떤 파일을 고칠지 정하도록 하려는 목적이었다.

이 방향은 맞았다.

하지만 map만으로는 부족했다.

왜냐하면 map은 "현재 구조를 설명"할 뿐, 좋은 구조를 강제하지는 않기 때문이다.

`view/home_screen.dart`가 1000줄이 되어도 map에는 "홈 화면"이라고 적을 수 있다.  
bridge가 export 파일이어도 map에는 "feature-view bridge"라고 적을 수 있다.

즉 map은 필요하지만, map만으로는 IBA가 작동하지 않았다.

구조 규칙이 더 구체적이어야 했다.

## 그래서 IBA를 다시 정의했다

이번 개편의 핵심은 "AI가 해석해야 하는 규칙"을 줄이는 것이었다.

처음 IBA는 이런 느낌이었다.

```text
섬끼리 직접 import 금지
bridge로 연결
기능 하나 = 파일 하나
```

말은 맞지만 충분히 구체적이지 않았다.

그래서 IBA v2에서는 기준을 다시 잡았다.

```text
파일 = 섬
최상위 책임 폴더 = 나라
하위 폴더 = 지역
bridge = 계약
map = 작업 지도
```

이 기준으로 전체 구조를 다시 정리했다.

## 의사결정 1: 모든 폴더가 나라가 아니다

이전 글에서는 `feature/`, `view/`, `viewmodel/` 같은 폴더를 섬처럼 설명한 부분이 있었다.

실제 적용 후 이 표현은 수정이 필요했다.

파일은 섬이다.

그리고 나라는 source-root 바로 아래의 최상위 책임 폴더다.

예를 들면 앱과 프론트엔드에서는 이런 폴더가 나라다.

```text
model/
feature/
viewmodel/
view/
bridge/
style/
external/
```

백엔드에서는 기준이 조금 다르다.

```text
model/
feature/
routes/
repository/
bridge/
config/
middleware/
external/
```

반대로 하위 폴더는 나라가 아니다. 지역이다.

```text
view/component/home/home_task_card.dart
feature/usecase/login/google_social_login.dart
feature/pure/date/date_dday_label_format.dart
```

이 파일 하나하나가 섬이다.

나라 안에는 같은 성격의 섬이 모이고, 하위 폴더는 그 섬을 찾기 위한 지역 역할을 한다.

```text
feature/
  pure/
  usecase/

view/
  screen/
  component/
```

이렇게 정의해야 AI가 "작업 단위는 파일이고, 큰 책임 경계는 최상위 폴더"라고 이해한다.

## 의사결정 2: bridge는 export가 아니라 contract다

이번 개편에서 가장 중요한 의사결정이다.

bridge는 export 파일이 아니다.

bridge는 나라 사이의 계약이다.

예를 들어 viewmodel-view bridge는 이런 식이어야 한다.

```text
bridge/viewmodel-view/home/home_view_contract.dart
```

그리고 이 파일에는 view가 받을 계약이 들어간다.

```dart
class HomeViewContract {
  final List<HomeTaskCardContract> taskCards;
  final HomeAppBarContract appBar;
  final VoidCallback onAddTask;

  HomeViewContract({
    required this.taskCards,
    required this.appBar,
    required this.onAddTask,
  });
}

class HomeTaskCardContract {
  final String title;
  final String ddayLabel;
  final VoidCallback onTap;

  HomeTaskCardContract({
    required this.title,
    required this.ddayLabel,
    required this.onTap,
  });
}
```

view는 `HomeViewContract`만 안다.  
viewmodel은 이 contract를 조립한다.  
feature의 계산 결과가 필요하면 viewmodel이 feature-viewmodel bridge를 통해 받아온다.

이렇게 해야 서로 모른다.

단순히 export로 우회하는 건 IBA가 아니다.

## 의사결정 3: bridge 단위를 체계화한다

한 번은 bridge를 파일마다 만들까 고민했다.

하지만 그러면 bridge가 폭발한다.

반대로 나라 전체를 하나의 bridge로 묶으면 연결은 단순해 보이지만 책임이 다시 흐려진다.

그래서 bridge 단위는 이렇게 정했다.

```text
feature area + country boundary
```

예를 들어 홈 화면이라면:

```text
bridge/viewmodel-view/home/home_view_contract.dart
bridge/feature-viewmodel/home/home_feature_contract.dart
```

로그인 외부 연동이라면:

```text
bridge/feature-external/login/login_external_contract.dart
```

섬마다 bridge를 만들지 않는다.  
그렇다고 나라 전체를 하나의 bridge로 묶지도 않는다.

기능 영역과 나라 경계를 기준으로 묶는다.

이 방식은 bridge 수를 최소화하는 전략이 아니다. 오히려 필요한 bridge는 늘어날 수 있다. 핵심은 수가 아니라 기준이다.

AI가 "이번에는 어떤 bridge를 만들어야 하지?"를 매번 새로 판단하지 않도록, bridge 생성 기준을 고정하는 것이 목적이었다.

## 의사결정 4: view는 screen과 component로 나눈다

실제 적용에서 가장 먼저 눈에 띈 문제가 `home_screen.dart` 비대화였다.

그래서 view 규칙을 명확히 했다.

```text
view/
  screen/
  component/
```

`screen/`은 조립자다.

```text
view/screen/home_screen.dart
```

screen은 화면 전체 흐름을 구성하고, 필요한 component를 배치한다.

반대로 `component/`는 UI 조각이다.

```text
view/component/home/home_app_bar.dart
view/component/home/home_task_card.dart
view/component/home/home_fab.dart
```

component는 렌더링만 한다.  
feature를 직접 모른다.  
viewmodel도 직접 모른다.

필요한 데이터와 callback은 contract로 받는다.

## 의사결정 5: feature는 pure와 usecase로 나눈다

기존 `feature/home/` 안에는 성격이 다른 코드가 섞였다.

그래서 feature도 둘로 나눴다.

```text
feature/
  pure/
  usecase/
```

`feature/pure/`는 순수 함수다.

```text
feature/pure/date/date_dday_label_format.dart
feature/pure/validation/string_email_validate.dart
feature/pure/task/list_subtask_progress_calculate.dart
```

여기에는 계산, 검증, 포맷팅, 파싱, 매핑이 들어간다.

`feature/usecase/`는 하나의 행동이다.

```text
feature/usecase/login/google_social_login.dart
feature/usecase/task/local_task_save.dart
```

로그인, 저장, 삭제, 동기화, 제출처럼 부작용이 있을 수 있는 행동은 usecase다.

이렇게 나누면 bridge에 mapper가 들어가는 문제도 줄어든다.

데이터 변환은 bridge가 아니라 `feature/pure/`에 둔다.

## 의사결정 6: model은 Data Shape이면 직접 import를 허용한다

처음에는 모든 직접 import를 금지하려고 했다.

그런데 model까지 bridge를 타게 하니 구조가 이상해졌다.

실제 프로젝트에서도 `feature-model`, `viewmodel-model` 같은 bridge가 생겼다. 하지만 대부분은 model을 re-export하는 파일이었다.

```dart
export 'package:app_project/model/mock_home_data.dart'
    show MockSubTask, MockTask, MockPage, kMockPages;
```

이건 불필요한 우회였다.

그래서 model의 역할을 줄이고 예외를 만들었다.

```text
model/은 Data Shape만 가진다.
Data Shape Model은 직접 import 가능하다.
```

대신 model에는 로직을 넣지 않는다.

금지:
- business logic
- UI logic
- API/DB/SDK call
- formatting
- validation
- mapping
- sorting/filtering/calculation

model이 멍청한 데이터 구조라면 직접 import해도 된다.

로직이 들어가야 한다면 `feature/pure/`나 `feature/usecase/`로 옮긴다.

## 의사결정 7: 문서 템플릿도 다시 나눴다

이번 문제는 코드 구조만의 문제가 아니었다.

AI 에이전트에게 작업을 시키려면 문서 흐름도 명확해야 한다.

기존에는 `templates/documents/`와 `templates/reports/`가 섞여 있었다.

그런데 작업하면서 보니 두 종류는 분리해야 했다.

프로젝트 산출물 템플릿:

```text
planning/
  PRD_TEMPLATE.md
  REQUIREMENTS_TEMPLATE.md
  USERFLOW_TEMPLATE.md
  DATA_MODEL_TEMPLATE.md
  API_SPEC_TEMPLATE.md
  TECH_DECISION_TEMPLATE.md
```

디자인 산출물 템플릿:

```text
design/
  screen/
    SCREEN_SPEC_TEMPLATE.md
    CLAUDE_DESIGN_PROMPT_TEMPLATE.md
  style/
    COLOR_TEMPLATE.md
    TYPOGRAPHY_TEMPLATE.md
    SPACING_TEMPLATE.md
    SHADOW_TEMPLATE.md
    BREAKPOINT_TEMPLATE.md
    COMPONENTS_TEMPLATE.md
    INTERACTION_TEMPLATE.md
```

리서치 산출물 템플릿:

```text
research/
  RESEARCH_BASE_TEMPLATE.md
  TECH_RESEARCH_MODULE.md
  MARKET_RESEARCH_MODULE.md
  COMPETITOR_RESEARCH_MODULE.md
  DATA_RESEARCH_MODULE.md
```

반대로 작업 진행 상황을 전달하기 위한 보고 형식은 템플릿에서 뺐다.

보고 형식은 프로젝트 산출물이 아니라 커뮤니케이션 규칙에 가깝기 때문이다. 그래서 템플릿은 실제로 저장되고 재사용되는 문서 산출물에만 집중하도록 정리했다.

## 기술 분석도 이름을 바꿨다

기술 분석이라는 말도 애매했다.

기술 분석 문서가 다루는 영역은 조사다.

```text
어떤 기술이 있는가?
각 기술의 특징과 장단점은 무엇인가?
실제 사용 사례는 무엇인가?
```

기술 결정 문서가 다루는 영역은 선택이다.

```text
우리 프로젝트에서는 무엇을 쓸 것인가?
왜 그 선택을 하는가?
무엇을 포기하는가?
```

그래서 둘을 나눴다.

```text
research/tech/tech_[topic].md
planning/tech-decision/tech_decision_[topic].md
```

조사와 결정을 섞으면 근거를 충분히 모으기 전에 결론이 먼저 고정될 수 있다.

그래서 조사 문서는 후보와 근거를 모으는 데 집중하고, 결정 문서는 선택과 포기한 대안을 기록하도록 분리했다.

## 개편 후 IBA 구조

개편된 앱 구조는 이렇게 잡았다.

```text
source-root/
  main.*
  app.*
  router.*

  model/
  feature/
    pure/
    usecase/
  viewmodel/
  view/
    screen/
    component/
  bridge/
    viewmodel-view/
    feature-viewmodel/
    feature-external/
  style/
  external/
```

백엔드는 이렇게 간다.

```text
backend-root/
  main.*

  model/
  feature/
    pure/
    usecase/
  routes/
  repository/
  bridge/
    routes-feature/
    feature-repository/
    feature-external/
  config/
  middleware/
  external/
```

핵심 규칙은 줄였다.

1. 파일이 섬이다.
2. 나라는 source-root 바로 아래의 최상위 책임 폴더다.
3. 하위 폴더는 나라가 아니라 지역이다.
4. 나라 간 연결은 bridge를 통한다.
5. 단, Data Shape Model과 read-only support country는 직접 import 가능하다.
6. view는 screen/component 지역으로 나눈다.
7. feature는 pure/usecase 지역으로 나눈다.
8. bridge는 export가 아니라 contract다.
9. mapper는 bridge가 아니라 feature/pure에 둔다.
10. 작업 전 `*-map.md`를 읽는다.
11. 구조 변경 후 `*-map.md`를 업데이트한다.

## 이번 개편의 핵심

처음 IBA는 "직접 import를 막고 bridge를 쓰자"에 가까웠다.

하지만 실제로는 그것만으로 부족했다.

AI는 폴더 이름만 보고 의도를 완벽히 이해하지 않는다.  
bridge 폴더가 있다고 해서 contract를 만들지도 않는다.  
view 폴더가 있다고 해서 component를 나누지도 않는다.

그래서 규칙을 더 구체적으로 만들었다.

| 실제 문제 | 개편 방향 |
|---|---|
| `home_screen.dart`에 UI가 몰림 | `view/screen`, `view/component` 분리 |
| bridge가 export 파일이 됨 | bridge는 contract/delegate만 허용 |
| model까지 bridge로 우회 | Data Shape Model은 직접 import 허용 |
| feature 안에 계산/행동이 섞임 | `feature/pure`, `feature/usecase` 분리 |
| 기술 조사와 기술 결정이 섞임 | `tech_research`, `tech_decision` 분리 |
| 문서 템플릿과 보고 형식이 섞임 | 산출물 템플릿과 커뮤니케이션 규칙 분리 |

## 마치며

이번에 느낀 건 단순하다.

아키텍처는 폴더 이름이 아니다.

AI에게 `bridge/`라는 폴더를 만들어준다고 해서, AI가 계약을 설계하지는 않는다.  
`view/`라는 폴더를 만들어준다고 해서, AI가 컴포넌트를 나누지도 않는다.

AI 친화적 아키텍처를 만들려면 의도를 구조로 고정해야 한다.

파일 하나의 책임, bridge의 역할, 예외 규칙, 문서 흐름, map 업데이트까지 모두 명시해야 한다.

설계는 선언이 아니라, 문제가 생긴 결과를 보고 다시 좁히는 일이다.
