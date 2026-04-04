---
status: "published"
title: "Hugo GitHub Pages 세팅 트러블슈팅"
date: "2026-04-04"
tags: ["트러블슈팅", "Hugo", "GitHub Pages", "GitHub Actions"]
---

Hugo + GitHub Pages 블로그 자동화 세팅 중 발생한 문제들과 해결 방법을 정리한다.

---

## 1. brew, node, gh 명령어를 찾지 못하는 문제

### 문제

터미널에서 `brew`, `node`, `gh` 등의 명령어를 실행하면 `command not found` 오류가 발생했다.

```
zsh: command not found: brew
zsh: command not found: node
```

### 원인

Homebrew와 관련 도구들이 `/opt/homebrew/bin/`에 설치되어 있었지만, 해당 경로가 `PATH` 환경변수에 포함되지 않은 상태였다. Claude Code의 Bash 실행 환경이 일반 터미널 세션과 다르게 초기화되어 `.zshrc`의 PATH 설정이 적용되지 않은 것이 원인이었다.

### 해결

명령어 실행 전 매번 PATH를 명시적으로 설정했다.

```bash
export PATH="/opt/homebrew/bin:$PATH" && brew install gh hugo
```

평소 터미널에서는 `.zshrc`에 아래 줄이 있으면 자동으로 적용된다.

```bash
export PATH="/opt/homebrew/bin:$PATH"
```

---

## 2. GitHub Pages 이미 활성화 오류 (HTTP 409)

### 문제

GitHub Pages를 활성화하는 API를 호출했을 때 아래 오류가 발생했다.

```
{"message":"GitHub Pages is already enabled.","status":"409"}
```

### 원인

GitHub 레포를 생성할 때 GitHub이 자동으로 GitHub Pages를 기본 활성화해두었다. 그 상태에서 다시 생성(POST) 요청을 보내 충돌이 발생했다.

### 해결

생성(POST) 대신 업데이트(PUT) 요청으로 GitHub Actions 빌드 방식으로 변경했다.

```bash
gh api repos/hyun471/hyun471.github.io/pages --method PUT -f build_type=workflow
```

---

## 3. GitHub Actions 빌드 실패 — 테마를 불러오지 못하는 문제

### 문제

로컬에서는 Hugo 빌드가 정상 동작했지만 GitHub Actions에서 빌드하면 테마를 찾지 못하고 실패했다.

### 원인

PaperMod 테마를 `git submodule`로 연결했는데, GitHub Actions의 `actions/checkout`은 기본적으로 submodule을 함께 받아오지 않는다. 그래서 `themes/PaperMod/` 폴더가 빈 상태로 빌드가 실행됐다.

### 해결

`actions/checkout` 단계에 `submodules: recursive` 옵션을 추가했다.

```yaml
- uses: actions/checkout@v4
  with:
    submodules: recursive   # 이 옵션이 없으면 테마 폴더가 비어서 빌드 실패
```

---

## 4. gh auth login 브라우저 인증 창을 실수로 닫은 경우

### 문제

`gh auth login` 실행 후 브라우저 인증 창이 열렸는데, 인증을 완료하기 전에 창을 닫았다. 터미널이 멈춰있고 로그인이 완료되지 않았다.

### 해결

터미널에서 `Ctrl + C`로 현재 프로세스를 종료하고 `gh auth login`을 다시 실행하면 새로운 인증 코드가 발급된다. 이전 코드는 자동으로 만료된다.

```bash
# Ctrl + C 후
gh auth login
```

---

## 5. node scripts/publish.js 실행 시 블로그 레포 경로를 찾지 못하는 경우

### 문제 상황

`publish.js`에서 블로그 레포 경로를 상대 경로로 지정했는데, 실행 위치에 따라 경로가 달라져 파일을 찾지 못할 수 있다.

### 해결

`__dirname`을 기준으로 절대 경로를 계산했다. `__dirname`은 현재 실행 중인 파일이 위치한 폴더의 절대 경로를 반환하므로 어디서 실행해도 항상 올바른 경로를 가리킨다.

```js
// __dirname = /Users/hs/Desktop/workspace/tech-blog-automation/scripts
const BLOG_DIR = path.join(__dirname, "../../hyun471.github.io/content/posts");
// 결과: /Users/hs/Desktop/workspace/hyun471.github.io/content/posts
```
