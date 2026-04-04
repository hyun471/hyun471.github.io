---
title: "노션 TIL을 벨로그에 자동 업로드하려다 실패한 이야기"
status: "published"
date: "2026-04-04"
type: "troubleshooting"
---

## 배경

꾸준히 작성해온 노션 TIL 페이지가 37개가 쌓였다. 취업 준비를 하면서 기술 블로그를 운영하는 것이 중요하다고 생각했고, 노션에 정리된 내용을 벨로그에 올리기로 했다.

문제는 37개를 하나씩 복사-붙여넣기 하기가 너무 번거롭다는 것이었다. 그래서 **자동화**를 시도했다.

---

## 1차 시도: Velog GraphQL API

벨로그는 `https://v2.velog.io/graphql` 엔드포인트를 사용한다. 개발자 도구로 확인하면 글 작성 시 `writePost` mutation을 사용한다는 것을 알 수 있다.

```javascript
mutation WritePost($input: WritePostInput!) {
  writePost(input: $input) {
    id
    url_slug
  }
}
```

벨로그 계정의 `access_token`을 쿠키에 담아 요청을 보냈다.

```javascript
const response = await fetch('https://v2.velog.io/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'cookie': `access_token=${process.env.VELOG_ACCESS_TOKEN}`,
  },
  body: JSON.stringify({ query: mutation, variables: { input } }),
});
```

### 결과: 실패

응답은 에러 없이 돌아왔지만 `data.writePost`가 `null`이었다. 여러 방법을 시도했다.

- `access_token` + `refresh_token` 쿠키 둘 다 전송 → 동일하게 `null`
- `origin`, `referer` 헤더 추가 → 동일
- `WritePostInput` 타입 없다는 것 확인 후 인라인 필드로 변경 → 동일

**결론:** v2 GraphQL API가 외부 접근을 차단하고 있는 것으로 판단했다. 공식적으로 외부 API 사용을 지원하지 않는 것 같다.

---

## 2차 시도: Playwright 브라우저 자동화

API가 막혀있다면 브라우저를 직접 조작하는 방법을 쓰면 된다. **Playwright**를 선택했다.

전략은 단순했다.
1. 벨로그에 로그인 → 세션 저장
2. 이후 헤드리스 모드로 글 작성 페이지 접근 → 자동 입력 → 출간

```bash
npm install playwright
npx playwright install chromium
```

### 문제 1: 구글 로그인 차단

벨로그에 구글 계정으로 가입했기 때문에 구글 OAuth를 거쳐야 했다. Playwright의 Chromium으로 로그인을 시도하자 구글이 **"자동화된 소프트웨어에 의한 접근"**으로 차단했다.

**시도한 해결책:**
- `channel: 'chrome'` 옵션으로 실제 Chrome 사용 → 여전히 차단
- Stealth 플러그인(`playwright-extra-plugin-stealth`) 도입 검토

### 문제 2: 대안 로그인 방법도 막힘

구글 로그인이 막히자 다른 방법을 찾아봤다.

| 방법 | 결과 |
|------|------|
| 벨로그 이메일 로그인(매직링크) | 이메일이 Gmail → Chromium에서 Gmail 접근 시 동일하게 구글 차단 |
| 네이버 이메일로 신규 가입 | 네이버 로그인 시 CAPTCHA 요구 |
| GitHub 로그인 | Google Authenticator(2FA) 요구 |

모든 로그인 경로가 막혔다. Stealth 플러그인으로 돌파하는 방법도 있었지만, 이 시점에서 **"자동화에 너무 많은 비용을 쏟고 있다"** 는 판단을 했다.

---

## 의사결정: 자동화 포기, 반자동화로 전환

자동화 실패의 근본 원인은 **벨로그가 외부 자동화를 막고 있다**는 것이다. API는 null을 반환하고, 브라우저 자동화는 구글/네이버/GitHub 인증에서 모두 차단됐다.

여기서 선택지는 두 가지였다.

1. Stealth 플러그인으로 계속 돌파 시도
2. 자동화 범위를 좁혀서 **할 수 있는 부분만 자동화**

37개 글을 올리는 게 목적인데, 자동화 문제 해결에 시간을 더 쓰는 것은 비효율적이라고 판단했다. **반자동화**로 방향을 바꿨다.

---

## 최종 해결책: 노션 → 마크다운 파일 추출

자동화할 수 있는 부분은 자동화했다.

**자동화한 것:**
- 노션 API로 TIL 페이지 전체 읽기
- 완성/미완성 판단 및 정리
- 마크다운 변환 + 이미지 로컬 다운로드

**수동으로 하는 것:**
- 벨로그에 마크다운 복붙
- 이미지 드래그앤드롭 업로드

```
output/
  01_변수 - 함수.md
  02_클래스.md
  ...
  34_Github 협업.md
  images/
    01_변수 - 함수/
      img_01.png
    ...
```

이미지도 페이지별 폴더에 번호순으로 저장되기 때문에, 벨로그에 글을 올릴 때 어떤 이미지가 어디 들어가는지 헷갈릴 일이 없다.

---

## 마무리

자동화를 완성하지 못했지만, 이 과정에서 얻은 것들이 있다.

- Velog API의 외부 접근 제한 파악
- Playwright를 활용한 브라우저 자동화 구조 이해
- 구글/네이버 등 OAuth 제공자의 봇 탐지 메커니즘 경험
- **"어디까지 자동화할 것인가"에 대한 의사결정 경험**

완벽한 자동화보다 **지금 당장 동작하는 것**이 더 가치 있다는 걸 다시 한번 느꼈다.
