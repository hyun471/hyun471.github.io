---
status: "published"
type: "posts"
title: "브릿지를 지우려다, usecase를 지웠다 — IBA 3차 개편"
date: "2026-07-09"
description: "브릿지를 계약으로 고친 뒤 앱에 더 밀어붙였더니 새 문제가 나왔다. 분석하고 평가하고, '브릿지가 꼭 필요한가'까지 되물은 끝에 내린 결론과, 그 과정에서 배운 것."
tags: ["아키텍처", "AI에이전트", "IBA", "리팩토링", "MVVM"]
---

## 들어가며

[지난 글](https://hyun471.github.io/posts/iba-ai-friendly-architecture-refactor/)에서 IBA(Island Bridge Architecture)를 실제 앱에 적용했다가 구조가 의도대로 안 나오는 걸 확인했다. 그때 가장 큰 문제는 브릿지가 `export`로 하위 계층을 그대로 재노출해서, 경계 역할을 전혀 못 하는 거였다. 그래서 **"브릿지는 export가 아니라 contract다"** 로 고쳤다. 브릿지가 계약(뷰가 받을 데이터·콜백의 형상)을 조립해서 넘기고, 내부 구현은 숨기는 구조로.

이번 글은 그 다음이다. 계약으로 고친 브릿지를 들고 화면을 계속 붙여나갔더니, 이번엔 다른 결의 문제가 올라왔다. 그리고 그 문제를 분석하다가, 결국 **"브릿지가 정말 필요한가"** 라는 근본 질문까지 되물었다. 그 재점검의 기록이다.

미리 결론을 말하면 — 나는 브릿지를 지우려다가, 정작 usecase를 지웠다.

## 문제 1 — 브릿지가 두 겹이 되면서 저장 한 번에 5단계

계약을 제대로 두려고 하니, 브릿지가 자연스럽게 두 종류로 갈렸다.

```text
bridge/
  feature-viewmodel/settings/settings_bridge.dart   # 기능 ↔ 뷰모델 경계
  viewmodel-view/settings/settings_bridge.dart       # 뷰모델 ↔ 뷰 경계
```

뷰모델이 기능을 부를 때 지나는 브릿지 하나, 뷰가 뷰모델을 볼 때 지나는 브릿지 하나. 논리적으로는 깔끔해 보였다. 그런데 실제로 "온보딩에서 선택한 플랜을 저장한다" 같은 아주 단순한 동작 하나가 이렇게 흘렀다.

```text
뷰(저장 탭)
  → bridge/viewmodel-view (계약의 onStart 콜백)
    → viewmodel (saveAndProceed)
      → bridge/feature-viewmodel (게이트웨이)
        → feature/usecase (실제 저장)
```

값 하나 저장하는 데 **5단계**. 파일 다섯 개를 열어야 흐름이 보였다. AI가 "저장이 왜 안 되지?"를 추적할 때 이 다섯 개를 순서대로 따라가야 했고, 나 역시 그랬다.

## 문제 2 — 브릿지가 남의 뷰모델을 직접 찔렀다

더 나쁜 건, 이 두 겹이 정작 경계를 못 지켰다는 거였다.

온보딩 저장이 성공하면, 저장 성공 여부와 별개로 **다른 전역 상태**(앱 시작 상태 = 라우팅을 결정하는 상태)를 갱신해야 했다. S1에서 완료 처리를 하면 앱이 S2로 넘어가야 하니까. 그런데 "한 뷰모델의 액션 결과가 다른 뷰모델을 어떻게 바꾸는가"는 스펙에 정의가 없었다.

정의가 없으니 AI는 그 순간 답을 만들어냈다. 온보딩 브릿지 안에서, 관계없는 앱시작 상태 provider를 직접 붙잡아 상태를 갈아끼우는 코드였다.

```dart
// bridge/viewmodel-view/onboarding/onboarding_bridge.dart
onStart: () async {
  final saved = await ref.read(onboardingProvider.notifier).saveAndProceed();
  if (!saved) return;

  // 온보딩 브릿지가 '앱시작 상태'를 직접 조작한다 — 경계 위반
  ref.read(appStartupStateProvider.notifier)
     .applyPersistedOnboardingCompletion(true);
}
```

브릿지의 존재 이유가 "경계를 지키는 것"인데, 그 브릿지가 경계를 넘는 편법의 온상이 됐다. 게다가 이건 문서로 금지돼 있던 것도 아니었다 — 애초에 이 상황에 대한 규칙 자체가 없었으니까.

여기에 더해, 지난 개편에서 분명히 금지했던 `export`도 다른 브릿지에 여전히 남아 있었다.

```dart
// bridge/viewmodel-view/home/home_bridge.dart
export 'package:my_app/feature/pure/home/list_s5_task_filter_sort.dart'
    show S5FilterChip, S5TaskItem;
export 'package:my_app/feature/pure/home/string_dday_urgency_calculate.dart'
    show computeDDayString;
```

금지 규칙이 문서에 있었고, 심지어 코드 리뷰 에이전트도 돌고 있었는데, 뚫렸다. 이건 나중에 다시 다룬다.

## 문제 3 — 전역 상태는 5개인데 뷰모델은 8개

전체를 훑어보다가 뷰모델 폴더에서 이걸 발견했다.

```text
viewmodel/
  home_viewmodel.dart          # 할일 목록 (구)
  task_list_viewmodel.dart     # 할일 목록 (신) ← 같은 상태를 관리
  settings_viewmodel.dart      # 사용자 정보 (구)
  user_viewmodel.dart          # 사용자 정보 (신) ← 같은 정보를 관리
  selected_task_viewmodel.dart # 선택된 Task ← 아무 화면도 안 씀
  app_startup_viewmodel.dart
  ai_plan_viewmodel.dart
  ...
```

설계서에는 전역 상태가 **5개**로 정해져 있었다. 그런데 뷰모델 파일은 **8개**였다. `home_viewmodel`과 `task_list_viewmodel`은 둘 다 "할일 목록"이라는 같은 상태를 관리하고 있었고, `settings_viewmodel`과 `user_viewmodel`은 둘 다 "사용자 정보"를 관리했다. 심지어 `selected_task_viewmodel`은 어느 화면도 소비하지 않는 완전한 죽은 코드였다.

원인은 단순했다. "뷰모델을 **화면당** 만드는지, **상태당** 만드는지" 정의가 없었다. 그래서 AI는 새 화면이 생길 때마다 뷰모델을 하나씩 찍어냈다. 상태 관리를 5개 단위로 새로 설계한 뒤에도, 옛 화면용 뷰모델이 안 지워진 채 공존했다.

## 분석 — 세 문제의 공통 뿌리

문제 1, 2, 3을 나란히 놓으니 원인이 하나로 모였다. 브릿지라는 **개념**이 아니라, 스펙의 **애매함**이었다.

| 문제 | 정의되지 않았던 것 | AI가 즉흥적으로 채운 방식 |
|---|---|---|
| 브릿지 2겹 5단계 | "브릿지를 무슨 단위로 만드나" | 경계마다 하나씩 → 두 겹 |
| 브릿지가 남의 뷰모델 찌름 | "뷰모델 간 상호작용을 어떻게 하나" | 브릿지가 직접 조작 |
| 뷰모델 8 vs 상태 5 | "뷰모델을 화면당? 상태당?" | 화면당 찍어냄 |

정의되지 않은 빈칸을, 매번 컨텍스트가 초기화되는 AI 세션이 각자 다르게 메꿨다. 그리고 그 서로 다른 해석들이 쌓여 구조를 갈라놓았다. 이건 지난 글의 교훈("아키텍처는 폴더 이름이 아니다")의 연장선이었다 — 폴더를 나눠도, 그 안에서 "몇 개를 어떤 기준으로 만드는지"가 비어 있으면 애매함은 그대로다.

## 그리고 껍데기 하나 — usecase

애매함과 별개로, 층 자체가 껍데기인 것도 있었다. `feature/usecase/` 폴더를 열어보니 파일이 13개였는데, 줄 수와 패턴을 세어봤다.

```text
openai_ai_plan_usecases.dart   282줄   ← 진짜 오케스트레이션 (API 호출+파싱+매핑)
update_user_profile_name.dart   56줄   ← 스냅샷 읽고 / 이름 바꾸고 / 저장
update_onboarding_completion.dart 56줄  ← 스냅샷 읽고 / 완료값 바꾸고 / 저장
save_onboarding_result.dart     61줄   ← 스냅샷 읽고 / 스타일 바꾸고 / 저장
update_user_planning_style.dart 55줄   ← 스냅샷 읽고 / 플랜 바꾸고 / 저장
... (나머지도 전부 동일 패턴)
```

**13개 중 12개가 "스냅샷 읽고 → 필드 하나 바꾸고 → 저장"** 똑같은 골격이었다. 각 50줄대의 거의 복붙. 실제로 두 파일을 나란히 놓으면 바뀌는 건 "어느 필드를 바꾸느냐" 한 줄뿐이었다.

진짜 다단계 로직은 `openai_ai_plan_usecases.dart`(282줄) 하나뿐. 나머지는 클린 아키텍처의 "액션 하나당 usecase 하나" 관성으로 AI가 기계적으로 찍어낸 껍데기였다. 오케스트레이션(읽고-바꾸고-쓰고 순서 엮기)은 이미 브릿지가 하고 있었으니, usecase는 그걸 한 번 더 감싼 빈 상자였다.

## 평가 — 무엇이 제 값을 하나

층별로 실제 하는 일을 저울에 올렸다.

**브릿지의 진짜 일**은 이거였다. 뷰가 액션을 트리거하고, 그 결과로 뷰모델을 갱신하는 걸, 뷰·기능·뷰모델이 서로를 직접 참조하지 않고 해내는 것. 이걸 없애면 누군가는 대신 해야 한다. 뷰가 기능을 직접 부르면 경계가 무너지고, 기능이 뷰모델을 갱신하면 의존 방향이 역전된다. 그래서 브릿지는 진짜 역할이 있었다.

**usecase의 실제 일**은 거의 없었다. 12개가 브릿지가 이미 하던 오케스트레이션을 한 겹 더 감싼 것뿐. 값을 하는 건 브릿지, 껍데기는 usecase였다.

## 재점검 — 브릿지가 꼭 필요한가?

그래도 나는 근본 질문을 던졌다. 여기까지 왔으니, 브릿지를 아예 없애고 순정 MVVM으로 가면 어떤가? 뷰모델이 기능을 직접 부르고, 상태를 갱신하고, 뷰는 뷰모델만 보면 되지 않나? 그러면 브릿지도 usecase도 없이 층이 넷으로 줄어든다.

감이 아니라 표로 따졌다.

| 기준 | 브릿지 제거 (MVVM) | 브릿지 유지 |
|---|---|---|
| 오케스트레이션 위치 | 뷰모델 (상태와 섞임) | 브릿지 (상태와 분리) |
| "저장 로직 고쳐줘" | 뷰모델을 건드림 | 브릿지만 건드림 |
| "상태 모양 바꿔줘" | 뷰모델을 건드림 (액션과 섞임) | 뷰모델만 |
| 크로스 상태 조율 | 뷰모델이 뷰모델 호출 (결합) | 브릿지가 중립에서 조율 |
| 층 수 | 최소 | 하나 더 |
| AI가 이미 아는가 | ✅ MVVM은 훈련데이터에 흔함 | ❌ 자체 설계, 매번 학습 |

MVVM의 약점이 뚜렷했다. 뷰모델이 상태와 액션을 겸하면 fat해지고, AI가 상태를 고치다 액션을 깨거나 그 반대가 나기 쉽다. "이 화면의 상태"와 "이 화면의 동작"이 한 파일에 뒤섞이니까.

반대로 브릿지를 유지하면 **"이 화면의 모든 액션은 이 브릿지 하나에 있다"** 는, grep 한 번에 잡히는 단일 타깃이 생긴다. AI가 수술적으로 편집할 때 이게 결정적이다. 애초에 IBA가 원했던 "수정 범위 제한"이 바로 이거였다.

그래서 결론은 뒤집혔다. **지울 건 브릿지가 아니라 usecase였다.** 그리고 브릿지의 실패는 개념이 아니라 애매함이었으니, 애매함만 제거하면 브릿지는 살아남는다. 물론 MVVM이 더 안전한 경우도 있다 — 자체 규칙을 강제할 여력이 없다면, AI가 이미 아는 MVVM이 낫다. 이건 트레이드오프지 정답이 아니다.

## 의사결정 1 — 브릿지는 화면당 하나, 호출만

브릿지를 살리되, 실패 원인이었던 애매함을 제거했다.

먼저 **"컨텍스트당 브릿지 하나"를 "화면당 브릿지 하나"로 바꿨다.** 컨텍스트가 뭔지는 또 헷갈렸지만, 화면은 셀 수 있고 안 헷갈린다. S1 → `s1_onboarding_bridge.dart`, S2 → `s2_todo_list_bridge.dart`. 브릿지 수 = 화면 수. "이게 한 컨텍스트냐 둘이냐" 하는 세션별 해석 차이가 사라진다.

그리고 **브릿지 메서드는 호출만 한다**로 못 박았다. 그 화면의 버튼·이벤트마다 메서드가 하나씩 생기는데, 메서드 안은 호출의 나열뿐이다.

```dart
// s8_settings_bridge.dart
Future<void> saveUserName(String name) async {
  final snapshot = await _storage.loadSettings();   // external: 읽기
  final now = nowIso();                             // impure: 현재 시각
  final next = applyName(snapshot, name, now);      // pure: 새 형상 만들기
  final ok = await _storage.saveSettings(next);     // external: 쓰기
  if (ok) {
    ref.read(userVM.notifier).setName(name);        // viewmodel: 상태 갱신
  } else {
    ref.read(userVM.notifier).setError(2001);
  }
}
```

시퀀싱·`await`·성공/실패 분기까지는 브릿지가 해도 된다. 하지만 **스스로 계산·검증·변환하는 순간 선을 넘은 것**이다. 그건 기능(pure)이나 뷰모델의 일이다. 문제 2에서 봤던 위반들(브릿지 안에서 D-Day 문자열 계산, 남의 상태 조작, export)이 정확히 이 선을 넘은 것들이었다.

크로스 상태 조율도 여기서 풀렸다. **트리거한 화면의 브릿지가 소유하고, 필요한 뷰모델들의 공개 메서드를 부른다.** 온보딩 완료는 S1에서 일어나니, S1 브릿지가 사용자 상태와 앱시작 상태 둘 다의 공개 메서드를 부른다. 뷰모델끼리 서로 찌르지 않는다.

```dart
// s1_onboarding_bridge.dart
Future<void> finishOnboarding(PlanStyle style) async {
  final ok = await _storage.saveOnboarding(style, nowIso());
  if (!ok) { ref.read(startupVM.notifier).setError(5001); return; }
  ref.read(userVM.notifier).setPlanStyle(style);       // 상태 1
  ref.read(startupVM.notifier).setCompleted(true);     // 상태 2 → 라우터가 반응
}
```

## 의사결정 2 — 기능을 pure / impure / external로

브릿지가 부르는 "기능"도 정리했다. 기준은 하나 — **같은 입력에 같은 출력이냐.**

- **pure**: 같은 입력 → 항상 같은 출력. (우선순위 점수 계산, 정렬, 검증)
- **impure**: 같은 입력 → 출력이 달라짐. 단, 앱 밖 경계는 안 넘음. (`DateTime.now()`, ID 생성)
- **external**: 앱 밖 경계를 넘음. (로컬 저장, API 호출) + 파싱/역파싱

impure를 따로 빼는 게 핵심이다. `now()`나 ID 생성을 격리하면, pure 함수들이 그걸 **인자로 받아서** 100% 결정적으로 유지된다. 실제로 우선순위 계산은 이미 이렇게 돼 있었다.

```dart
// feature/pure/home/score_priority_calculate.dart
// today를 인자로 받으므로 pure — 파일 안에 DateTime.now() 가 0개
int? computeDueDateScore(String? dueDate, DateTime today) { ... }
int? computePriorityScore({ required int importance, required int? dueScore }) { ... }
```

`DateTime.now()`를 밖(브릿지)에서 읽어 `today`로 주입받은 덕분에, "오늘 기준 D-Day"라는 시간 의존 계산이 pure로 남았다. 그래서 테스트에서 `today`에 `2026-07-09`를 박아 검증할 수 있다. 만약 이 함수가 안에서 `now()`를 읽었다면 impure가 됐을 것이고, 매 실행마다 결과가 달라져 테스트가 불가능해졌을 것이다.

## 의사결정 3 — 모델을 entity / dto로

전에 로컬 저장 포맷을 snake_case로 바꿨다가, 앱을 켜자마자 전역이 파싱 에러로 죽은 적이 있다. 원인은 저장 포맷(외부)과 앱 내부 모델이 안 나뉘어 있어서였다. 저장 JSON을 읽는 코드가 이런 식이었다.

```dart
// created_at 키가 없으면 그대로 크래시
createdAt: json['created_at'] as String,
```

시뮬레이터에 남아 있던 구버전 데이터에는 `createdAt`(camelCase)만 있고 `created_at`이 없었다. `null as String`이 런타임 예외를 던졌고, 그게 저장 로직 전체를 무너뜨렸다.

그래서 model을 둘로 쪼갰다.

- **entity**: 앱 내부에서 쓰는 형상. `copyWith`만 갖는다.
- **dto**: 외부 형상(저장 JSON, API 응답). `fromJson/toJson/copyWith`를 갖고, **없는 키를 절대 그냥 캐스팅하지 않는다.**

```dart
// model/dto/task_dto.dart — 방어적 파싱
factory TaskDto.fromJson(Map<String, dynamic> j) => TaskDto(
  id: (j['id'] as String?) ?? '',
  title: (j['title'] as String?) ?? '',
  status: (j['status'] as String?) ?? 'incomplete',
);
```

이러면 저장 포맷이 바뀌어도 dto와 변환(external에 둔다)만 고치면 되고, entity와 그걸 쓰는 화면은 안 건드린다. 포맷 변경의 폭발 반경이 dto 하나로 갇힌다.

## 의사결정 4 — 규칙은 문서가 아니라 린트로

가장 크게 배운 건 이거다. 문제 2에서 본 브릿지 `export` 위반은 **문서에 명확히 금지돼 있었고, 코드 리뷰 에이전트까지 돌고 있었는데 뚫렸다.** LLM 리뷰어는 확률적이라 100%가 안 된다. 그리고 사후에 잡는다.

그래서 강제를 성격에 따라 둘로 나눴다.

**기계로 막을 수 있는 건 린트로.**
- import 방향 규칙 (뷰는 기능 import 금지 등)
- 브릿지에 `export` 금지
- 안 쓰는 라우트 금지
- 브릿지가 계산 라이브러리 import 금지 (호출만 강제)

어기면 리뷰 코멘트가 아니라 **빌드 실패**다. 뚫렸던 `export`가 바로 이 종류였다 — 확률적인 리뷰어가 아니라, 결정적인 린트가 잡았어야 했다.

**정적으로 못 잡는 의미 규칙만 리뷰어에게.**
- "브릿지에 로직 없음", "뷰모델에 I/O 없음", "impure 격리됨"

단, 일반 판단에 맡기지 않고 층별 체크리스트로 준다. 그리고 이 핵심 규칙들은 매 dev 에이전트 태스크의 프롬프트에 직접 주입한다 — "필요하면 이 문서를 읽어라"는 안 통한다는 걸 이미 확인했으니까.

## 문제 ↔ 의사결정 매핑

| 문제 | 원인 | 의사결정 |
|---|---|---|
| 브릿지 2겹, 저장 5단계 | 브릿지 단위 미정의 | 화면당 브릿지 하나 |
| 브릿지가 남의 뷰모델 조작 | 크로스 상태 규칙 없음 | 트리거 화면 브릿지가 소유·조율 |
| 뷰모델 8 vs 상태 5 | 뷰모델 단위 미정의 | 전역 상태당 뷰모델 하나 |
| usecase 12개 껍데기 | 클린 아키텍처 관성 | usecase 폐기, 브릿지가 오케스트레이션 |
| 파싱 크래시 | 저장 포맷과 모델 미분리 | model entity/dto 분리 + 방어적 파싱 |
| export가 리뷰어를 뚫음 | 문서 규칙에 의존 | 린트로 강제, 리뷰어는 의미 규칙만 |

## 마치며

브릿지를 지우려 했는데, 정작 지운 건 usecase였다. 이게 이번 개편의 한 줄 요약이다. 브릿지의 역할(뷰·기능·뷰모델을 커플링 없이 잇기)은 진짜였고, usecase는 그걸 감싼 빈 상자였다. "무엇을 지울까"를 감이 아니라 "각 층이 실제로 뭘 하나"로 따졌더니 답이 뒤집혔다.

그리고 지난 글과 똑같은 자리로 다시 돌아왔다. 지난번엔 "아키텍처는 폴더 이름이 아니다"였는데, 이번엔 한 걸음 더 갔다. **좋은 규칙을 만드는 것보다, 그 규칙이 지켜지게 만드는 게 어렵다.** 문서에 "브릿지는 export 금지"라고 아무리 또박또박 써도, 그게 문서로만 있으면 어겨진다. 리뷰어를 붙여도 확률적으로 샌다. 결국 **문서가 아니라 린트로 내릴 수 있는 규칙이 몇 개냐가 아키텍처의 진짜 수명을 정한다.**

마지막으로, 이 개편을 준비하며 마주친 반전 하나. 코드에서 죽은 뷰모델과 죽은 기능이 잔뜩 나왔는데, 뜯어보니 그게 **설계서가 지정한 올바른 목표 구조**였다. 새 상태 관리를 다 만들어놓고 화면 연결만 안 해서, 잘 만든 새 코드가 죽은 채 방치돼 있었고 화면은 옛 코드로 돌고 있었다. "죽은 코드를 지우자"고 그냥 달려들었으면, 지워야 할 옛 코드는 남기고 살려야 할 새 코드를 지울 뻔했다.

설계는 선언이 아니라, 문제가 생긴 결과를 보고 다시 좁히는 일이라는 걸 또 배운다. 이번에도 방향은 "더 정교하게"가 아니라 "덜어내고, 지켜지게"였다. 그게 이 구조가 아직 살아있다는 증거이길 바란다.
