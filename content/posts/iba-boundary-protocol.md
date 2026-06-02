---
status: "published"
type: "posts"
title: "Island Bridge Architecture 보완기: AI가 지킬 수 있는 경계 만들기"
date: "2026-06-03"
description: "IBA를 실제 개발 흐름에 맞게 다듬으며 용어, 브릿지, 계약, 작업 프로토콜을 다시 정리한 기록"
tags: ["architecture", "IBA", "AI", "refactoring", "protocol"]
---

# Island Bridge Architecture 보완기: AI가 지킬 수 있는 경계 만들기

이전 글에서 Island Bridge Architecture, 줄여서 IBA를 소개했다.

IBA는 AI가 코드를 작성할 때 불필요한 파일 탐색과 범위 밖 수정을 줄이기 위해 만든 구조다. 핵심은 파일을 독립된 섬처럼 다루고, 경계를 넘는 연결은 bridge로 명시하는 것이다.

그런데 개념을 설명하는 것과 실제 개발 흐름에서 AI가 그 구조를 정확히 지키게 만드는 것은 다른 문제였다.

구조는 있었지만 해석이 흔들렸다. bridge가 export처럼 쓰이거나, contract와 gateway의 역할이 섞이거나, 작업 전에 어느 파일까지 봐야 하는지 명확하지 않은 상황이 생겼다.

이번 글은 IBA의 개념 소개가 아니라, 실제 적용 과정에서 IBA를 어떻게 보완했는지에 대한 의사결정 기록이다.

## 1. 먼저 용어를 다시 정리했다

가장 먼저 해야 할 일은 용어 정리였다.

처음에는 섬, 다리, 문 같은 표현을 썼지만, 실제 코드 구조로 옮기면 애매한 부분이 생겼다.

폴더가 섬인가?

파일이 섬인가?

bridge는 섬과 섬을 잇는가, 나라와 나라를 잇는가?

contract는 bridge인가, bridge를 통과하는 데이터인가?

이 질문들이 정리되지 않으면 AI도 같은 지점에서 헷갈린다.

그래서 IBA의 용어를 이렇게 다시 정리했다.

```text
Country = 최상위 책임 폴더
Region = 나라 안의 역할/기능/화면 단위 구역
Island = 파일 하나
Bridge Gateway = 나라 경계를 넘는 region/context 단위 출입구
Contract = gateway를 통과하는 데이터/콜백 규격
Map = 섬과 브릿지의 위치를 알려주는 작업 지도
```

핵심은 이것이다.

```text
파일은 섬이다.
섬과 섬은 연결될 수 있다.
단, 서로 다른 나라의 섬끼리 연결하려면 bridge gateway가 필요하다.
bridge gateway는 섬마다 만들지 않고, region/context 단위로 설치한다.
```

예를 들어 App/Frontend에서는 이런 식이다.

```text
view/
  screen/
    home/
      home_screen.dart

  component/
    home/
      home_task_card.dart

feature/
  usecase/
    social_login/
      google_social_login.dart
  pure/
    social_login/
      login_error_message_key_resolve.dart

bridge/
  viewmodel-view/
    home/
      home_bridge.dart
      home_contract/
        home_screen_contract.dart
        home_task_card_contract.dart
```

여기서 `view/`, `feature/`, `bridge/`는 나라다.

`screen/home`, `component/home`, `usecase/social_login`은 지역이다.

각 `.dart` 파일은 섬이다.

그리고 `home_bridge.dart`는 섬마다 하나씩 만들어지는 다리가 아니라, `home`이라는 region/context에서 나라 경계를 넘는 출입구다.

이 정리가 중요했다.

왜냐하면 IBA의 목적은 멋진 비유가 아니라, AI가 "어디까지 봐야 하고, 어디부터 보면 안 되는지"를 판단하게 만드는 것이기 때문이다.

## 2. bridge를 "계약 자체"에서 "출입구 + 계약" 구조로 바꿨다

처음에는 bridge와 contract의 역할이 섞여 있었다.

bridge가 곧 계약처럼 쓰였다. 어떤 bridge 파일에는 class가 있고, 어떤 파일에는 enum이 있고, 어떤 파일은 export만 하고, 어떤 파일은 함수처럼 동작했다.

이렇게 되면 AI가 bridge를 읽어도 알기 어렵다.

이 파일이 연결 지점인지, 통과 데이터인지, 실제 로직인지, 단순 export인지 구분이 흐려진다.

그래서 bridge를 두 개념으로 나눴다.

```text
Bridge Gateway = 출입구
Contract = 출입구를 통과할 수 있는 데이터/콜백 규격
```

예를 들면:

```text
bridge/viewmodel-view/home/
  home_bridge.dart
  home_contract/
    home_screen_contract.dart
    home_task_card_contract.dart
```

`home_bridge.dart`는 출입구다.

view와 viewmodel 사이에서 무엇을 가져오고 어떤 callback을 연결할지 제어한다.

`home_screen_contract.dart`와 `home_task_card_contract.dart`는 통과 가능한 데이터 모양이다.

view는 이 contract만 알고, viewmodel 내부 구현은 모른다.

이렇게 분리하면 AI가 이해해야 할 질문이 단순해진다.

```text
어디가 연결 지점인가? -> bridge gateway
무엇이 통과하는가? -> contract
실제 로직은 어디 있는가? -> 각 island
```

이전에는 bridge 하나가 여러 역할을 섞어 가질 수 있었다.

이제는 gateway와 contract의 책임이 나뉘었기 때문에 경계가 더 명확해졌다.

## 3. 작업 전 프로토콜을 만들었다

IBA는 폴더 구조만으로 완성되지 않는다.

폴더가 잘 나뉘어 있어도 AI는 작업 중에 더 많은 파일을 열어보고, 관련 있어 보이는 파일을 수정하고, "이것도 같이 고치면 좋겠다"고 판단할 수 있다.

그래서 작업 전에 반드시 경계를 확정하는 프로토콜이 필요했다.

신규 기능이나 화면을 만들기 전에는 먼저 확인한다.

```text
1. 어떤 country가 관련되는가
2. 어떤 region/context에서 작업하는가
3. 어떤 island를 만들거나 수정하는가
4. 어떤 bridge gateway가 필요한가
5. 어떤 contract가 필요한가
6. map 파일에 어떤 변경이 필요한가
7. 개발 허용 파일 목록은 무엇인가
8. 그 목록 밖 파일 수정 금지
```

수정 작업도 마찬가지다.

```text
1. 수정 대상 island 확인
2. 관련 bridge gateway 확인
3. 관련 contract 확인
4. boundary 변경 여부 판단
5. boundary가 그대로면 해당 island만 수정
6. boundary가 바뀌면 gateway/contract와 직접 연결된 섬만 수정
7. map 업데이트 여부 확인
```

이 프로토콜의 목적은 개발 절차를 복잡하게 만드는 것이 아니다.

목적은 하나다.

**AI가 작업을 시작하기 전에 개발 경계를 확정하게 만드는 것.**

## 4. 프로토콜은 개발 절차가 아니라 개발 경계 확정 장치다

프로토콜을 만든 이유는 IBA의 목적과 직접 연결된다.

IBA는 단순히 폴더를 예쁘게 나누는 구조가 아니다. AI가 불필요한 파일을 읽지 않게 하고, 개발하지 않아도 되는 파일을 건드리지 않게 하는 구조다.

그러려면 작업 전에 경계가 정해져야 한다.

예를 들어 홈 화면 카드 UI만 수정하는 작업이라면 AI는 다음만 보면 된다.

```text
view/component/home/home_task_card.dart
bridge/viewmodel-view/home/home_bridge.dart
bridge/viewmodel-view/home/home_contract/home_task_card_contract.dart
app-map.md
```

반대로 소셜 로그인 기능의 SDK 호출 방식만 바뀐다면 볼 파일은 달라진다.

```text
feature/external/social_login/google_social_login_trigger.dart
feature/usecase/social_login/google_social_login.dart
bridge/feature-viewmodel/home/home_bridge.dart
app-map.md
```

프로토콜은 이 차이를 작업 전에 정리하게 만든다.

즉, 프로토콜은 "어떻게 개발할 것인가"보다 "어디까지 개발할 것인가"를 정하는 장치다.

이게 없으면 AI는 IBA 구조를 알고도 작업 중에 범위를 넓힐 수 있다. 이게 있으면 AI는 개발 허용 파일 목록 밖으로 나가기 어렵다.

## 5. 결과적으로 IBA는 이렇게 바뀌었다

처음 IBA는 이런 느낌이었다.

```text
섬끼리 직접 import 금지
bridge로 연결
기능 하나 = 파일 하나
```

이제는 이렇게 정리된다.

```text
Country = 최상위 책임 폴더
Region = 나라 안의 역할/기능/화면 단위 구역
Island = 파일 하나
Bridge Gateway = 나라 경계를 넘는 region/context 단위 출입구
Contract = gateway를 통과하는 데이터/콜백 규격
Protocol = 작업 전 개발 경계 확정 절차
```

그리고 핵심 규칙은 이것이다.

```text
파일은 섬이다.
섬과 섬은 연결될 수 있다.
서로 다른 나라의 섬끼리 연결하려면 bridge gateway가 필요하다.
bridge gateway는 섬마다 만들지 않고 region/context 단위로 설치한다.
contract는 gateway를 통과하는 데이터와 callback만 정의한다.
작업 전 protocol로 개발 가능한 island, gateway, contract, map을 확정한다.
```

이번 보완에서 가장 크게 바뀐 건 폴더 이름이 아니다.

AI가 구조를 해석하는 방식이다.

이전에는 AI에게 "bridge를 써라"라고 말하는 수준이었다.

이제는 AI에게 "어떤 파일이 섬이고, 어떤 경계를 넘을 때 출입구가 필요하며, 어떤 데이터만 통과할 수 있고, 작업 전 어디까지 개발할 수 있는지 확정하라"고 말한다.

결국 IBA에서 중요한 것은 bridge 폴더 자체가 아니다.

중요한 것은 **AI가 지킬 수 있는 개발 경계**다.

## 마치며

이번 보완을 하면서 느낀 것은 하나다.

AI 친화적 아키텍처는 사람이 보기 좋은 구조만으로는 부족하다.

사람은 맥락을 읽고 적당히 조절할 수 있지만, AI는 명시되지 않은 기준을 자주 다르게 해석한다.

그래서 AI에게 맡길 구조라면 용어, 경계, 예외, 작업 순서까지 더 구체적으로 고정해야 한다.

IBA는 이제 "섬과 다리"라는 비유에서 조금 더 실용적인 구조가 되었다.
