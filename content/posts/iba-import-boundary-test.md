---
status: "published"
type: "posts"
title: "아키텍처 규칙을 린트가 아니라 테스트로 강제했다"
date: "2026-09-21"
description: "AI는 규칙 문서를 읽어도 어긴다. import 경계를 테스트로 만들어 flutter test에서 깨지게 한 과정."
tags: ["IBA", "아키텍처", "테스트", "Flutter", "AI"]
---

# 아키텍처 규칙을 린트가 아니라 테스트로 강제했다

IBA를 설계하고 문서로 정리했다. 계층별로 무엇을 import할 수 있는지 표까지 만들어뒀다. 그런데 AI한테 코드를 맡기면 계속 어겼다.

몇 번 지적하다가 생각을 바꿨다. 규칙 문서가 있어도 어기는 건, 어겼을 때 아무 일도 안 일어나기 때문이다.

## 왜 린트가 아니었나

처음엔 custom_lint로 만들려고 했다. 그런데 검사해야 할 게 단순한 import 금지가 아니었다.

- 계층마다 허용 대상 집합이 다르다. view는 bridge 추상체만, bridge impl은 feature와 viewmodel
- 화면 단위로 짝이 맞아야 한다. `s2_bridge_impl.dart`는 `s2_bridge.dart`만 구현해야 하고 다른 화면 브릿지를 건드리면 안 된다
- `model`과 `common`은 누구나 import 가능이라는 예외가 있다

룰 엔진에 얹는 것보다 파일 경로를 파싱해서 판정하는 코드를 직접 쓰는 게 빨랐다. 그래서 테스트로 갔다.

```yaml
# analysis_options.yaml
# IBA import boundary rules are not lint rules here —
# they are enforced by test/iba_import_boundary_test.dart, run via `flutter test`.
```

나중에 헷갈릴까 봐 설정 파일에 주석으로 남겨뒀다.

## 어떻게 검사하나

`lib/` 아래 모든 `.dart` 파일의 import를 파싱해서 세 단계로 본다.

### 1. 경로를 계층으로 분류한다

```dart
class _Layer {
  final String name;
  final String? screenId;   // 화면 단위 매칭용
  const _Layer(this.name, [this.screenId]);
}
```

`view/screens/s1_onboarding_screen.dart`는 `viewScreens` + screenId `s1_onboarding`,
`bridge/impl/s1_onboarding_bridge_impl.dart`는 `bridgeImpl` + 같은 screenId로 분류된다.

screenId를 같이 들고 다니는 게 포인트다. 계층만 보면 "브릿지 impl이 브릿지 추상체를 import했다"까지밖에 못 본다. 어느 화면 건지는 모른다.

### 2. 허용 집합과 대조한다

```dart
const Map<String, Set<String>> _allowedTargets = {
  'viewScreens': {
    'model', 'common', 'viewComponent',
    'viewScreenWidget', 'bridgeAbstraction',
  },
  // ...
};
```

문서의 표를 그대로 코드로 옮겼다. 표가 바뀌면 이 맵도 바꾼다.

### 3. 화면 짝을 확인한다

브릿지 추상체와 impl이 같은 screenId인지 따로 검사한다.

## 결과

`flutter test` 한 번이면 끝난다. 규칙을 어기면 테스트가 깨진다. AI가 "이게 더 편한데요" 하면서 지름길을 내도 그 자리에서 잡힌다.

지금 import 위반은 0건이다. 룰을 잘 지켜서가 아니라 어기면 못 넘어가서다.

## 아직 못 잡는 것

테스트가 보는 건 import 방향뿐이다. 이런 건 못 잡는다.

- 브릿지 impl이 몇 줄인지. 비대해지는 건 규칙 위반이 아니다
- viewmodel에 특정 화면 전용 상태가 들어갔는지
- model 아래 entity가 몇 개까지 늘었는지

세 번째가 지금 제일 문제다. entity가 45개쯤 됐는데 성격이 섞여 있다. 이것도 세는 방법을 찾는 중이다.

## 마치며

AI한테 지키라고 말하는 것과 지킬 수밖에 없게 만드는 건 다르다. 문서만 있을 때는 계속 어겨졌고, 테스트를 넣고 나서 멈췄다.

규칙을 만들 때 어겼을 때 무슨 일이 일어나는지도 같이 만들었어야 했다. 그 사이에 쌓인 위반을 걷어내는 데 시간이 더 들었다.
