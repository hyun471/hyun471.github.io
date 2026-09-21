---
status: "published"
type: "troubleshooting"
title: "언어를 바꿨는데 페이지 일부만 바뀐다"
date: "2026-09-21"
description: "Flutter가 그리는 캔버스 밖 HTML은 앱 상태를 모른다. 커스텀 이벤트로 양쪽 언어를 맞춘 기록."
tags: ["FlutterWeb", "다국어", "l10n", "JS interop", "트러블슈팅"]
---

# 언어를 바꿨는데 페이지 일부만 바뀐다

올해 초 웹 프로젝트에서 다국어를 붙이다 만난 문제다. Flutter Web이 화면을 어떻게 그리는지 제대로 이해하게 된 계기이기도 하다.

## 환경

- Flutter Web
- ARB 기반 l10n
- `index.html`에 직접 쓴 정적 영역(소개, 푸터, 정책 링크)

## 문제 상황

언어를 한국어에서 영어로 바꿨다. Flutter가 그린 화면은 바뀐다. 그런데 소개, 푸터, 정책 링크는 한국어 그대로다.

그리고 새로고침하면 선택한 언어가 날아가고 기본값으로 돌아간다.

## 원인

### 정적 영역은 Flutter 밖이다

Flutter Web은 캔버스 위에 화면을 그린다. `index.html`에 직접 써둔 HTML은 그 캔버스 바깥에 있다. 앱의 상태 변경이 거기까지 안 간다.

앱에서 `locale`을 바꿔도 HTML 쪽은 자기가 바뀌어야 한다는 걸 모른다. 같은 페이지에 있어도 렌더링 주체가 달라서 상태가 공유되지 않는다.

### 언어 설정을 메모리에만 뒀다

저장을 안 해서 새로고침하면 사라졌다. 웹에서 새로고침은 앱 재시작이다.

## 해결

### 1. 언어를 저장하고 시작할 때 읽는다

```dart
// 저장
SessionStorage.set('locale', code);

// 앱 시작 시 읽어서 초기 언어로 쓴다
final saved = SessionStorage.get('locale');
```

### 2. 언어 변경을 커스텀 이벤트로 알린다

```dart
void setLocale(String code) {
  state = Locale(code);
  SessionStorage.set('locale', code);
  web.window.dispatchEvent(web.Event('locale-changed'));
}
```

### 3. HTML 쪽에서 듣는다

```html
<script>
  window.addEventListener('locale-changed', () => {
    const code = sessionStorage.getItem('locale') || 'ko';
    document.documentElement.lang = code;
    applyStaticTexts(code);   // 정적 영역 문구 교체
  });
</script>
```

`document.documentElement.lang`도 같이 바꾼다. 보이는 문구만 바꾸고 문서 언어 정보를 안 바꾸면 안 맞는다.

이벤트에 값을 실어 보내지 않고 스토리지를 읽게 한 데는 이유가 있다. 페이로드로 넘기면 Dart와 JS 사이 타입 문제를 또 다뤄야 한다. 값은 스토리지에 두고 이벤트는 "바뀌었다"는 신호만 보내니까 경계가 단순해졌다.

## 결과

- 언어를 바꾸면 Flutter 화면과 정적 HTML이 같이 바뀐다
- 새로고침해도 선택한 언어가 유지된다
- `lang` 속성이 실제 언어와 맞는다

## 마치며

Flutter Web에서 캔버스 밖 영역은 앱 상태를 모른다. 같은 페이지처럼 보여도 다른 세계다.

두 쪽을 이을 때는 값을 넘기는 것보다, 값은 공유 저장소에 두고 신호만 주고받는 게 편했다. 경계를 넘는 데이터가 적을수록 손볼 자리도 적다.
