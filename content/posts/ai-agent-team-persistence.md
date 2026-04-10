---
status: "published"
title: "Claude Code로 AI 에이전트 팀을 만들었는데, 대화가 끊긴다?"
date: "2026-04-09"
description: "Claude Code에서 AI 에이전트 팀을 구성할 때 발생하는 세션 지속성 문제를 세 가지 방법으로 탐구하고, CLAUDE.md와 서브에이전트를 결합한 현실적인 해결책을 소개한다."
tags: ["ClaudeCode", "AI에이전트", "LLM", "개발환경", "자동화"]
type: "posts"
---

# Claude Code로 AI 에이전트 팀을 만들었는데, 대화가 끊긴다?

## 배경: 왜 AI 에이전트 팀을 만들었나

Claude Code를 사용하다 보면 하나의 AI에게 모든 걸 맡기는 한계를 느끼게 된다. 기획, 설계, 개발, 검토까지 한 모델이 처리하면 역할이 섞이면서 결과물의 품질이 일정 수준 이상 올라가지 않는다.

그래서 실제 팀처럼 역할을 나눈 AI 에이전트 시스템을 설계했다.

```
PO (나)
└── PM
    ├── PlanLeader → PlanMember1, PlanMember2
    ├── DesignLeader → DesignMember1
    └── DevLeader → DevFront, DevBack, DevApp
```

각 에이전트는 `~/.claude/agents/` 폴더에 `.md` 파일로 정의했다. 역할, 사용 가능한 툴, 커뮤니케이션 규칙, 산출물 형식까지 구체적으로 명시했다. 이렇게 하면 어떤 프로젝트에서 Claude Code를 열어도 이 에이전트들이 자동으로 활성화된다.

---

## 문제: PM이 단발성이다

설계는 잘 됐는데, 실제로 써보니 핵심적인 문제가 있었다.

**`@PM`으로 호출하면 PM은 한 턴만 처리하고 사라진다.**

Claude Code에서 `@agent명`으로 서브에이전트를 호출하면 그 에이전트는 해당 요청만 처리하고 컨텍스트가 종료된다. 다음 메시지를 보내면 PM은 이전 대화를 전혀 기억하지 못한다.

PM과 회의를 하고, 태스크를 논의하고, 승인을 받고, 진행 상황을 보고받는 구조인데 — 대화가 이어지지 않으면 의미가 없다.

---

## 시도 1: Claude Agent SDK

지속적인 에이전트 팀을 만드는 방법을 찾다가 **Claude Agent SDK**를 발견했다.

### 어떤 건가
Python/TypeScript 코드로 에이전트를 프로그래밍하고, 세션 ID를 저장해 언제든 이어서 실행할 수 있는 방식이다.

```python
# 세션 저장
async for message in query(prompt="...", options=options):
    if isinstance(message, ResultMessage):
        save_session(message.session_id)

# 나중에 이어서
options = ClaudeAgentOptions(resume=saved_session_id)
```

세션은 `~/.claude/projects/.../*.jsonl`에 디스크에 저장되어 프로세스를 재시작해도 완전히 복원된다.

### 왜 포기했나
**API 키가 별도로 필요하고 토큰당 과금이 발생한다.**

Claude Code CLI는 Pro/Max 구독에 포함되지만, Agent SDK는 Anthropic API 키가 필요하다. 공식 문서에 명시되어 있다:

> "Set your API key — Get an API key from the Console"

Pro 구독($20/월) 외에 API 사용료가 추가로 발생한다. 에이전트 팀을 돌리면 토큰 소비가 일반 사용의 5~7배 수준이라 비용 부담이 크다.

---

## 시도 2: Claude Code 내장 Agent Teams

Claude Code에는 실험적 기능으로 **Agent Teams**가 내장되어 있다.

### 어떤 건가
`settings.json`에 환경변수 하나를 추가하면 활성화된다.

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

팀원들이 공유 태스크 리스트와 메일박스로 소통하며 협업한다. 기존 `~/.claude/agents/` 파일을 teammate 정의로 그대로 재활용할 수 있다.

### 왜 완전한 해결책이 아닌가
공식 문서에 명시된 한계가 있다:

1. **세션 복원 불가**: 종료 후 재시작하면 팀원들이 모두 사라진다
2. **중첩 팀 불가**: "No nested teams — teammates cannot spawn their own teams" — DevLeader가 DevFront를 teammate로 spawn할 수 없다
3. **실험적 기능**: 아직 불안정할 수 있다

PM → 팀장 → 팀원의 계층 구조를 완전히 구현할 수 없다는 게 핵심 문제다.

---

## 최종 해결: CLAUDE.md + 서브에이전트 하이브리드

### 핵심 아이디어
**`@PM`으로 호출하는 게 아니라, 메인 Claude Code 세션 자체가 PM이 되면 된다.**

프로젝트 폴더에 `CLAUDE.md`를 만들면 Claude Code가 세션 시작 시 자동으로 읽는다. 여기에 PM 역할을 지정하면 별도 호출 없이 세션 내내 PM으로 동작한다.

```markdown
# CLAUDE.md

You are the PM of this project.
Read ~/.claude/agents/PM.md and follow all rules defined there.
```

### 최종 아키텍처

```
메인 세션 = PM (CLAUDE.md로 세션 내내 지속)
    ↓ 작업 필요 시 서브에이전트 호출
DevLeader / PlanLeader / DesignLeader
    ↓ 필요한 팀원 자동 호출
DevFront, DevBack, PlanMember1/2, DesignMember1
```

### 왜 이 구조인가

**PM 지속성**: CLAUDE.md로 메인 세션이 PM이 되어 `claude --resume`으로 이전 세션을 이어갈 수 있다.

**팀장/팀원 효율성**: 서브에이전트는 작업 단위로 호출되어 불필요한 토큰 낭비가 없다. 각자 시작 시 `PROJECT-FOUNDATION.md`, map 파일, 로그를 읽어 컨텍스트를 복원한다.

**파일이 곧 메모리**: 세션이 끊겨도 파일 시스템이 기억을 대신한다.
- `meetings/` → 회의록, pm-inbox.md (팀장 보고 누적)
- `dev/logs/`, `planning/logs/`, `design/logs/` → 작업 이력
- `PROJECT-FOUNDATION.md` → 프로젝트 전체 컨텍스트

**Agent Teams는 선택적으로**: 팀장 간 실시간 소통이 필요한 복잡한 작업이 생기면 그때 사용한다. 같은 `~/.claude/agents/` 파일이 서브에이전트와 teammate 양쪽으로 동작하기 때문에 별도 설정 변경 없이 전환 가능하다.

---

## 결론

| 방법 | 지속성 | 계층 구조 | 추가 비용 | 안정성 |
|------|--------|-----------|-----------|--------|
| 서브에이전트만 | 없음 | 완전 지원 | 없음 | 안정 |
| Agent SDK | 완전 | 완전 지원 | API 과금 | 안정 |
| Agent Teams | 세션 내 | 중첩 불가 | 없음 | 실험적 |
| **CLAUDE.md + 서브에이전트** | **세션 내** | **완전 지원** | **없음** | **안정** |

완벽한 해결책은 없다. Agent SDK가 가장 강력하지만 비용이 따른다. 현실적인 선택은 CLAUDE.md로 PM 지속성을 확보하고, 파일 기반 메모리로 세션 간 컨텍스트를 복원하며, 필요할 때 Agent Teams를 선택적으로 쓰는 하이브리드 구조다.

AI 에이전트 팀을 설계할 때 가장 중요한 건 "어떻게 기억을 유지할 것인가"다. 그 답이 꼭 세션 지속성일 필요는 없다. 파일 시스템을 메모리로 활용하는 것도 충분히 실용적인 해결책이다.
