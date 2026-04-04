---
status: "published"
title: "Hugo + GitHub Actions로 마크다운 자동 배포 블로그 만들기"
date: "2026-04-04"
tags: ["Hugo", "GitHub Pages", "GitHub Actions", "Node.js", "자동화"]
---

# Hugo + GitHub Actions로 마크다운 자동 배포 블로그 만들기

공부한 내용을 블로그에 올리는 과정을 자동화하고 싶었다. 목표는 마크다운 파일을 저장하면 검토 후 명령어 한 줄로 블로그에 배포되는 시스템이다. 이 글은 그 시스템을 구성하는 기술들을 정리한 것이다.

---

## 1. Hugo — 정적 사이트 생성기

Hugo는 마크다운 파일을 HTML로 변환해주는 정적 사이트 생성기다. Go 언어로 만들어져 빌드 속도가 매우 빠르다.

### 핵심 구조

```
hyun471.github.io/
├── content/posts/   ← 마크다운 파일 위치 (여기에 추가하면 글이 생김)
├── themes/PaperMod/ ← 블로그 디자인 테마
├── hugo.toml        ← 전체 설정 파일
└── public/          ← 빌드 결과물 (HTML/CSS)
```

`content/posts/` 안에 마크다운 파일을 추가하면 Hugo가 이를 HTML 페이지로 변환한다. 직접 HTML을 건드릴 필요가 없다.

### Frontmatter

마크다운 파일 상단에 메타데이터를 작성하는 방식이다. Hugo는 이를 읽어 제목, 날짜, 태그 등을 자동으로 처리한다.

```markdown
---
title: "글 제목"
date: "2026-04-04"
tags: ["Flutter", "기초"]
---

본문 내용...
```

### Git Submodule로 테마 관리

PaperMod 테마를 설치할 때 직접 복사하지 않고 `git submodule`을 사용했다.

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

Submodule은 외부 Git 레포를 내 레포 안에 참조로 연결하는 방식이다. 테마 원본이 업데이트되면 `git submodule update --remote`로 쉽게 반영할 수 있다. 직접 복사하면 업데이트가 번거롭고 불필요한 파일이 레포에 쌓인다.

---

## 2. GitHub Pages — 무료 정적 호스팅

GitHub에서 제공하는 무료 웹 호스팅이다. 레포 이름을 `{유저명}.github.io`로 만들면 자동으로 `https://{유저명}.github.io` 주소로 접근할 수 있다.

별도 서버 비용이 없고, git push만으로 배포가 가능하다.

---

## 3. GitHub Actions — 자동 빌드/배포 파이프라인

`main` 브랜치에 push가 발생하면 GitHub Actions가 자동으로 Hugo 빌드를 실행하고 GitHub Pages에 배포한다.

```yaml
# .github/workflows/deploy.yml

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive      # 테마(submodule)도 같이 받아옴
      - uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "latest"
      - run: hugo --minify           # HTML/CSS 빌드 + 용량 최소화
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    steps:
      - uses: actions/deploy-pages@v4
```

`submodules: recursive` 옵션이 핵심이다. 이 옵션이 없으면 PaperMod 테마를 못 불러와 빌드가 실패한다.

---

## 4. Node.js 자동화 스크립트

마크다운 파일을 읽고, 블로그 레포로 복사하고, git push까지 실행하는 스크립트를 Node.js로 작성했다.

### 사용한 내장 모듈

**`fs` (File System)** — 파일 읽기, 복사, 삭제, 폴더 생성

```js
fs.readFileSync(filePath, "utf-8")   // 파일 읽기
fs.writeFileSync(destPath, content)  // 파일 쓰기
fs.copyFileSync(src, dest)           // 파일 복사
fs.unlinkSync(filePath)              // 파일 삭제
fs.mkdirSync(dir, { recursive: true }) // 폴더 생성
```

**`path`** — 운영체제에 관계없이 안전한 경로 처리

```js
path.join(__dirname, "../todo-list") // 상대 경로를 절대 경로로
```

**`child_process.execSync`** — Node.js에서 터미널 명령어 실행

```js
const { execSync } = require("child_process");

execSync(`git -C "${blogRepoDir}" add content/posts/`);
execSync(`git -C "${blogRepoDir}" commit -m "Add posts: ${fileName}"`);
execSync(`git -C "${blogRepoDir}" push`);
```

`git -C "경로"` 옵션은 해당 경로의 git 레포에서 명령을 실행하는 방식이다. `cd` 없이 특정 폴더의 git 작업을 수행할 수 있다.

### Frontmatter 자동 추가

업로드 시 `status: published`를 자동으로 삽입하는 로직이다. 파일이 이미 `---` frontmatter를 가지고 있으면 그 안에 추가하고, 없으면 새로 생성한다.

```js
function addFrontmatter(content, extra = {}) {
  const hasFrontmatter = content.startsWith("---");

  if (hasFrontmatter) {
    const end = content.indexOf("---", 3);
    const frontmatter = content.slice(0, end + 3);
    const body = content.slice(end + 3);
    const extraLines = Object.entries(extra)
      .map(([k, v]) => `${k}: "${v}"`)
      .join("\n");
    return frontmatter.replace("---\n", `---\n${extraLines}\n`) + body;
  }
  // frontmatter 없으면 새로 생성
  ...
}
```

---

## 전체 흐름 요약

```
마크다운 파일 작성
    ↓
todo-list/ 폴더에 저장
    ↓
node scripts/check.js → Claude가 완성/미완성 판단
    ↓
node scripts/publish.js --complete 파일명.md
    ↓
content/posts/ 복사 → git push
    ↓
GitHub Actions: Hugo 빌드 → GitHub Pages 배포 (약 30초)
    ↓
https://hyun471.github.io 에 글 반영
```

마크다운 작성과 배포를 완전히 분리한 구조 덕분에, 글 쓰는 흐름을 끊지 않고 필요할 때만 업로드할 수 있다.
