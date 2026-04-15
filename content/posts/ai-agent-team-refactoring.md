---
status: "published"
type: "posts"
title: "Claude Code 멀티 에이전트 팀, 직접 설계하고 부수고 다시 고쳤다"
date: "2026-04-15"
description: "AI 에이전트 팀을 직접 설계하고 실제로 돌려보니 25개의 문제가 쏟아졌다. 논리 모순, 권한 충돌, 중복 규칙까지 — 부수면서 배운 것들을 정리했다."
tags: ["ClaudeCode", "AI에이전트", "멀티에이전트", "IBA", "시스템설계"]
---

# Claude Code 멀티 에이전트 팀, 직접 설계하고 부수고 다시 고쳤다

Claude Code의 서브에이전트 기능으로 개발 프로젝트를 자동화하는 팀을 만들었다. PM, 개발자, 리서처, 디자이너, 코드 리뷰어까지 역할을 나누고 서로 보고하고 호출하는 구조. 설계할 땐 그럴싸해 보였다. 실제로 돌려보니 문제가 쏟아졌다.

---

## 배경: 에이전트는 명세서대로 정확하게 움직인다

사람이라면 "pubspec.yaml이 없는데 초기화 태스크니까 일단 해보자"라고 유연하게 판단한다. 에이전트는 Pre-check가 실패하면 그냥 멈춘다. 이게 단점처럼 보이지만 사실 장점이기도 하다. **명세가 정확하면 항상 같은 결과가 나온다.** 설계자가 논리를 정확하게 작성해야 한다는 책임이 생기는 것이다.

팀 구성은 다음과 같다:

| 에이전트 | 역할 |
|---------|------|
| PM | PO와 소통, 모든 서브에이전트 조율 |
| Researcher | 시장 조사, 기술 분석 |
| Verifier | Researcher 결과물 신뢰성 검증 |
| DevApp | Flutter 앱 개발 |
| DevFront | React 웹 프론트엔드 개발 |
| DevBack | FastAPI 백엔드 개발 |
| CodeReviewer | 코드 품질, 보안, 아키텍처 리뷰 |
| Designer | 디자인 시스템, 화면 스펙 작성 |
| Architect | 복잡한 기술 의사결정 자문 |
| Blogger | 블로그 게시물 발행 |

PM이 유일한 허브다. PO는 PM하고만 이야기하고, PM이 나머지 에이전트를 호출한다. 코드 구조는 **IBA(Island Bridge Architecture)** — 각 폴더를 독립적인 섬으로 취급하고, 섬끼리 직접 import를 금지하며 반드시 `bridge/`를 통해서만 연결하는 방식이다. AI 에이전트가 불필요한 파일을 탐색하거나 범위 외 파일을 수정하는 걸 구조적으로 차단하기 위한 선택이었다.

---

## Pre-check가 자기 자신을 막고 있었다

DevApp, DevFront, DevBack의 시작 흐름이 이렇게 설계되어 있었다:

```
Pre-check 1 — dev/IBA.md 존재 확인
Pre-check 2 — pubspec.yaml 존재 확인
↓
If this subtask IS the project initialization task → 초기화 실행
```

초기화 태스크일 때 pubspec.yaml이 당연히 없다. Pre-check 2가 먼저 실행되면서 "초기화가 안 됐으니 먼저 초기화 서브태스크를 요청하세요"라고 PM에게 보고하고 멈춰버린다. PM은 이미 초기화 서브태스크를 요청했는데.

순서를 바꿨다:

```
Step 1 — dev/IBA.md 존재 확인 (항상 필요)
Step 2 — 이 서브태스크가 초기화 태스크인가?
         Yes → Pre-check 2 건너뛰고 초기화 실행
         No  → Step 3으로
Step 3 — pubspec.yaml 존재 확인 (일반 작업에만 적용)
```

초기화 여부 판단을 Pre-check 2 앞으로 끌어올렸다. PM이 "초기화 서브태스크"로 명시해서 호출하면, 에이전트는 Pre-check 2를 건너뛰고 바로 초기화를 실행한다.

---

## 명세가 자기 자신과 모순될 때

가장 황당한 버그는 같은 파일 안에 있었다. PM이 관리하는 `PROJECT-FOUNDATION.md`에서:

- **Section 14**: Weekly Schedule (주간 일정)
- **Section 15**: Project Initialization Status (프로젝트 초기화 상태)

Project Initialization Check 로직:
> "Read PROJECT-FOUNDATION.md and check **Section 14**. Project Initialization Status"

Rule #18:
> "immediately update PROJECT-FOUNDATION.md **Section 15** (Project Initialization Status)"

**읽을 땐 14, 쓸 땐 15.** 실제로 동작했다면 항상 잘못된 섹션을 읽고 엉뚱한 판단을 했을 것이다.

비슷한 유형이 두 개 더 있었다. Verifier의 Minor 처리 규칙:
> "신뢰도가 낮은 출처는 연구 결과물 파일에 직접 `[신뢰도 낮음]`이라고 표시하라"

Verifier의 툴 목록은 `WebSearch, WebFetch, Read`다. **Write 툴이 없어서 파일에 아무것도 쓸 수 없다.**

Designer의 책임:
> "Execute design QA after dev team implementation"

Work Principles:
> "Only access the `design/` folder. Never access `dev/`"

개발 구현 결과물을 QA 하려면 `dev/` 폴더를 봐야 한다. 근데 접근이 금지되어 있다. 논리적으로 불가능한 책임을 가지고 있었다.

세 경우 모두 같은 원칙으로 해결했다. Section 참조는 일관되게 수정, Verifier와 Designer는 "보고만 하는" 역할로 정리 — Write 툴 추가 대신 책임 자체를 명확히 했다.

---

## 제거하거나, 통합하거나

### Planner 에이전트 제거

초기 설계엔 Planner가 있었다. PM이 PO와 회의를 마친 후 Planner를 호출해서 서브태스크 분해를 맡기는 구조.

PO의 요구사항, 우선순위, 맥락을 가장 잘 아는 건 그 대화를 진행한 PM이다. 그 내용을 Planner에게 다시 전달해서 분해를 맡기는 건 중간 단계만 늘리는 것이었다. PM → Planner → PM → PO보다 PM이 직접 분해하고 PO에게 보여주는 게 자연스럽다. Planner 제거.

### 서브태스크 테이블 형식 통일

PM.md 안에 서브태스크 관련 테이블이 세 곳에 있었고 형식이 모두 달랐다:

- **Subtask Breakdown Format**: `# | Subtask | Agent | Dependencies | Approach`
- **PO 승인 요청**: `Task | Subtask | 접근방법 | 담당에이전트 | 의존성 | 완료여부`
- **회의록**: `Task | Subtask | 접근방법 | 담당에이전트 | 의존성 | 완료여부 | 특이사항`

목적에 따라 두 가지로 정리했다. PO 승인 요청은 계획 확인용이므로 실행 추적 컬럼 불필요. 회의록은 실행 중 상태 업데이트가 필요하므로 전체 컬럼 유지. Subtask Breakdown Format은 PO 승인 요청과 같은 역할이어서 삭제.

### 폴더 초기화 타이밍 명확화

dev 에이전트를 처음 호출할 때 `dev/IBA.md`가 없으면 에이전트가 멈추고 PM에게 생성을 요청한다. 그런데 PM은 언제 이 폴더들을 만들어야 하는지 정의가 없었다.

Closing 절차에 "실행 전 환경 준비" 단계를 명시적으로 추가했다:

- Dev 에이전트 포함 시: `dev/`, `dev/IBA.md` 생성
- Researcher 포함 시: `planning/` 폴더 구조 생성
- Designer 포함 시: `design/` 폴더 구조 생성

각 에이전트 하위 폴더(`dev/app/`, `dev/frontend/`, `dev/backend/`)는 에이전트 자신이 초기화 태스크에서 직접 만든다. PM은 공용 영역만 책임진다.

---

## 디테일이 터지는 곳들

**Flutter 테스트 폴더 경로**: `dev/app/tests/`로 설정되어 있었다. `flutter test`는 기본적으로 `test/` (s 없음)를 바라본다. `tests/`로 만들면 별도 설정 없이는 테스트 러너가 파일을 인식하지 못한다. `dev/app/test/`로 수정. DevFront와 DevBack은 `tests/`도 허용되므로 유지 — 언어/프레임워크 표준을 따르는 게 원칙.

**앱 이름 전달 누락**: DevApp이 `flutter create --project-name [appname] .`를 실행하는데 앱 이름을 어디서 가져오는지 명시가 없었다. PM이 DevApp에 초기화를 요청할 때 `PROJECT-FOUNDATION.md Section 1`의 Service Name을 사용하고, 미정이면 Service Alias를 사용하도록 명시했다. 임시명으로 만들어진 경우를 위해 배포 전 체크리스트도 추가했다.

**CodeReviewer에 Grep 툴 없음**: IBA 준수 여부 검토의 핵심은 직접 import 패턴 탐지다. 어떤 파일이 `bridge/`를 거치지 않고 다른 island를 직접 import하는지 찾아야 한다. 툴이 `Read, Glob, WebSearch, WebFetch`뿐이었다. 파일 전체를 읽고 수동으로 찾아야 하는 상황. Grep 툴 추가.

**화면 스펙 폴더 분리**: Designer가 화면 스펙을 `design/output/screens/[screen-name].md`에 저장하고 있었다. DevApp과 DevFront가 같은 폴더를 참조하는데, 모바일 앱과 웹 화면 스펙이 섞여있으면 어떤 스펙을 참조해야 하는지 모호하다. `design/output/screens/app/`과 `design/output/screens/web/`으로 분리했다.

---

## 수정 결과

| 분류 | 건수 |
|------|------|
| Critical (플로우 차단) | 5건 |
| Warning (잠재적 오류) | 8건 |
| Minor (불일치/중복) | 12건 |
| **총계** | **25건** |

---

## 마치며

중복 내용이 생각보다 많았다. 같은 규칙이 세 곳에 있으면 나중에 하나만 수정하다가 불일치가 생긴다. 에이전트는 모든 내용을 읽고 판단하기 때문에, 모순된 지침이 있으면 예측하기 어려운 방향으로 동작한다. **단일 출처(single source of truth) 원칙이 AI 에이전트 명세에서도 똑같이 중요하다.**

앞으로 실제 프로젝트에 투입해볼 예정이다. 명세는 실전에서 더 많이 깨진다.
