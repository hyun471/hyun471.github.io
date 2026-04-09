---
title: "Island Bridge Architecture"
date: "2026-04-09"
description: "AI 에이전트의 토큰 효율과 작업 정확도를 극대화하기 위해 직접 설계한 아키텍처 패턴"
tags: ["아키텍처", "AI", "설계", "토큰최적화"]
type: "posts"
status: "published"
---

# Island Bridge Architecture (IBA)
## AI 친화적 아키텍처를 직접 설계하다

---

## 탄생 배경

기존 아키텍처들 — Clean Architecture, MVVM, Hexagonal — 은 모두 **사람이 읽고 유지보수하는 것**을 목표로 설계됐다.

하지만 AI 에이전트가 코드를 작성하는 시대가 됐다. AI는 사람과 다르게 동작한다.

- 파일을 읽을 때마다 **토큰을 소비**한다
- 연관 파일을 찾기 위해 **디렉토리를 탐색**하는 데도 토큰이 든다
- 파일 간 의존 관계가 복잡하면 **어디를 수정해야 할지 추론**하는 데 또 토큰이 든다

즉, 아키텍처가 복잡할수록 AI의 토큰 비용이 올라가고 실수 가능성도 높아진다.

**Island Bridge Architecture는 AI 에이전트의 토큰 효율과 작업 정확도를 최대화하기 위해 설계한 아키텍처다.**

---

## 핵심 개념

### 섬 (Island)
각 파일은 독립된 섬이다. 섬은 자기 역할만 한다. 다른 섬을 직접 알지 못한다.

```
model/      🏝️ 데이터 구조만 정의
feature/    🏝️ 처리 로직만 담당
viewmodel/  🏝️ 상태 관리만 담당
view/       🏝️ UI 렌더링만 담당
style/      🏝️ 시스템 설정만 담당
```

### 다리 (Bridge)
섬과 섬을 잇는 유일한 연결 통로. 직접 import 금지 — 반드시 bridge/를 통해서만 연결한다.

```
bridge/
├── feature-model/       ← feature ↔ model 연결
├── feature-viewmodel/   ← feature ↔ viewmodel 연결
├── feature-view/        ← feature ↔ view 연결
├── feature-style/       ← feature ↔ style 연결
└── viewmodel-view/      ← viewmodel ↔ view 연결
```

### 문 (Door)
bridge 폴더 안의 각 파일이 "문"이다. 특정 기능과 특정 섬을 이어주는 단일 연결점.

```
bridge/feature-view/
  local_email_login_door   ← 로그인 기능과 로그인 화면을 잇는 문
  kakao_oauth_login_door   ← 카카오 로그인 기능과 소셜 로그인 화면을 잇는 문
```

---

## 전체 디렉토리 구조

```
project/
├── model/               🏝️ 데이터 구조 (엔티티, DTO)
├── feature/             🏝️ 모든 처리 로직 — 기능 하나 = 파일 하나
├── viewmodel/           🏝️ 상태 관리
├── view/                🏝️ UI 요소 (위젯, 화면, 애니메이션)
├── style/               🏝️ 시스템 설정 (색상, 폰트, 언어)
└── bridge/
    ├── feature-model/
    ├── feature-viewmodel/
    ├── feature-view/
    ├── feature-style/
    └── viewmodel-view/
```

---

## 예시: 비밀번호 확인 기능

기존 방식이라면 `feature/password_check`가 `view/signup_page`를 직접 import한다. 둘이 서로를 안다.

IBA에서는:

```
feature/password_check
    ↓
bridge/feature-view/password_check_door
    ↓
view/signup_page
```

`feature/password_check`와 `view/signup_page`는 서로를 모른다. 문(door)만 통해서 연결된다.

**이 구조의 효과:**
- `view/signup_page`를 수정해도 `feature/password_check`는 영향 없음
- `feature/password_check`를 교체해도 `view/signup_page`는 영향 없음
- `bridge/feature-view/password_check_door`만 수정하면 연결이 바뀜

---

## 변경의 전파 범위

```
feature 구현만 바뀜  →  door 그대로  →  view 영향 없음  ✓
feature 계약이 바뀜  →  door 수정    →  view도 수정 (명시적으로)  ✓
```

변경이 **암묵적으로 퍼지지 않는다.** door가 바뀌면 연결된 섬도 고쳐야 한다는 게 명확하게 보인다.

---

## AI 친화적인 이유

### 1. bridge/만 읽으면 전체 연결 구조 파악
AI가 "이 기능이 어떤 화면과 연결됐지?"를 알려면 `bridge/feature-view/`만 읽으면 된다. 모든 파일을 탐색할 필요 없다. **토큰 절약.**

### 2. 파일명이 문서 역할
```
local_email_login.dart     ← 이메일 로컬 로그인
kakao_oauth_login.dart     ← 카카오 OAuth 로그인
access_token_issue.dart    ← 액세스 토큰 발급
refresh_token_renew.dart   ← 리프레시 토큰 갱신
```
파일 목록만 봐도 내용을 예측할 수 있다. AI가 파일을 열어보지 않아도 된다. **토큰 절약.**

### 3. 수정 범위가 명확
AI가 기능을 수정할 때 "어디까지 고쳐야 하나?"를 bridge/를 보고 바로 판단할 수 있다. **추론 비용 절약.**

### 4. AI의 작업 범위를 구조적으로 제한

IBA의 가장 중요한 특성이다.

섬C를 수정하는 작업이라면:

```
섬A (password_check) → bridge → 섬C (signup_page)
```

- ✅ 섬C 수정
- ✅ bridge 수정 (연결 변경 필요시)
- ❌ 섬A — 접근할 이유가 없다

기존 구조에서는 섬C를 수정하다가 AI가 "섬A도 조금만 고치면 더 좋을 것 같은데..." 하면서 범위를 넘어서는 순간 코드가 꼬이기 시작한다. 의도하지 않은 변경이 다른 기능에 영향을 미치고, 디버깅이 어려워진다.

IBA에서는 bridge가 **작업 경계선** 역할을 한다. 섬C 작업이면 섬C와 관련 bridge만 보면 된다. 섬A는 볼 필요도 없고, 건드려서도 안 된다. 구조 자체가 AI의 작업 범위를 명확히 제한한다.

### 5. 의존성 역전 불가능
섬끼리 직접 import가 금지되어 있으니 AI가 실수로 잘못된 방향으로 의존성을 만들 수 없다. **구조적 오류 방지.**

---

## 기존 아키텍처와 비교

| | MVVM | Clean Architecture | Hexagonal | **IBA** |
|--|--|--|--|--|
| 설계 목표 | 관심사 분리 | 의존성 역전 | 외부/내부 분리 | **AI 토큰 효율** |
| 연결 방식 | 직접 의존 | 인터페이스 | Port/Adapter | **Bridge/Door** |
| 연결 가시성 | 낮음 | 중간 | 중간 | **높음** |
| AI 친화성 | 낮음 | 중간 | 중간 | **높음** |
| 학습 곡선 | 낮음 | 높음 | 높음 | **중간** |

---

## IBA가 기존 패턴과 다른 점

- **Mediator 패턴**: 중앙 하나에 모든 연결이 집중 → God Object 문제 발생
- **Hexagonal의 Port**: 내부/외부 두 방향만 분리
- **IBA의 Bridge**: 레이어 쌍마다 bridge 폴더를 분리 → God Object 없이 연결 관계를 폴더 구조로 명시적으로 표현

연결 관계를 **폴더 구조 자체로 표현**한 것이 IBA의 독창적인 부분이다.

---

## 핵심 규칙 요약

1. **기능 하나 = 파일 하나** — 하나의 파일에 두 가지 기능 금지
2. **섬끼리 직접 import 금지** — 반드시 bridge/를 통해서만 연결
3. **파일명에 방식/범위/대상 명시** — `local_email_login.dart` (O), `login.dart` (X)
4. **파일 설명은 단일 기능으로** — "및", "와", "+" 포함 시 파일 분리 필요
5. **bridge/가 연결 지도** — bridge/ 읽으면 전체 연결 구조 파악 가능

---

*Island Bridge Architecture — 섬처럼 독립적으로, 다리로 연결되게.*
