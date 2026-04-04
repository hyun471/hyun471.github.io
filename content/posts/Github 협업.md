---
title: "Github 협업"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 Github 사용법 (with VSCode)

**기본 세팅**

**사용자 정보 설정**

- 터미널에 명령어로 사용자 정보를 입력하여 설정
- user.name : 커밋할 때 작성자로 기록될 이름
- user.email : 커밋할 때 작성자로 기록될 이메일
- -- global : PC 전체에서 동일하게 사용
```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

- 다음 명령어로 입력값 확인
```bash
git config --list
```

**GitHub 로그인**

- 사용자 설정이 완료된 이후 VSCode에서 Publish to GitHub를 클릭
![](/images/34_Github 협업/img_01.png)

- 클릭하면 GitHub 브라우저가 열리며 GitHub 계정 로그인
- 로그인 하고 “Authorize Visual Studio Code” 승인하면 VSCode에 GitHub계정이 연결됨
## 1인 개발

**1인 개발 repository 사용 흐름**

**github repository 생성**

**VSCode에서 바로 만들기**

- VSCode에서 Publish to GitHub를 클릭
- Repository의 이름을 정한 뒤 private과 public를 선택하여 생성
![](/images/34_Github 협업/img_02.png)

**터미널로 만들기**

- 터미널에서 명령어를 사용하여 로컬에 새 Git 레포 만들기
```bash
git init
```

- 터미널에서 명령어를 사용하여 현재 폴더의 모든 파일을 스테이징에 추가 
```bash
git add .
```

- 터미널에서 명령어를 사용하여 스테이징된 파일을 커밋 (커밋 시 메세지 작성 필수)
```bash
git commit -m "첫 커밋"
```

- GitHub 웹에서 새 Repository 만들기
![](/images/34_Github 협업/img_03.png)

![](/images/34_Github 협업/img_04.png)

-  터미널을 이용하여 웹에서 생성된 Repository의 주소를 넣고 로컬과 연결
```bash
git remote add origin https://github.com/username/repo.git
// 주소를 넣고 뒤에 꼭 .git 붙이기
```

- 현재 로컬 브랜치를 repo에 있는 브랜치 이름으로 강제 변경 
```bash
git branch -M main
```

- 터미널을 이용하여 로컬에 있는 브랜치를 repo의 업로드(첫 push는 다음과 같이 진행)
```bash
git push -u origin main
```

**github에 repository 생성 후 commit & push**

**VSCode**

- Commit 시 메서지 필수로 입력 후 Commit 버튼 클릭 (메서지는 Commit 규칙 참고)
![](/images/34_Github 협업/img_05.png)

- Push 시 Sync Changes 버튼 클릭(메세지는 선택)
![](/images/34_Github 협업/img_06.png)

**터미널**

- 모든 변경 사항을 스테이징에 올리기 
```bash
git add .
```

- Commit 메세지를 필수로 입력 후 스테이징을 로컬에 저장 (메서지는 Commit 규칙 참고)
```bash
git commit -m "변경 내용 설명"
```

- Repository에 로컬 브랜치를 Push 
```bash
git push
```

**Commit 메세지 규칙**

- 커밋 메세지 규칙의 경우 어떤 걸 했는지 쉽게 알기 위해 메세지 앞에 feature: 등을 붙임
- 암묵적으로 사용되지만 조직에 따라 커밋 메세지 규칙이 다를 수 있음
## 협업 시 Github 사용

### 팀장일 경우

**협업을 위한 팀장의 기본 세팅**

1. 프로젝트 생성 및 레포지토리 연결 (1인 개발의 레포지토리 생성과 동일한 과정)
1. main 브랜치 하위에 develop 브랜치 생성 (main 브랜치에는 배포 전에 병합)
![](/images/34_Github 협업/img_07.png)

![](/images/34_Github 협업/img_08.png)

![](/images/34_Github 협업/img_09.png)

1. github에서 기본 브랜치 세팅을 main에서 develop으로 변경 (병합 시 develop에 병합되도록 설정)
![](/images/34_Github 협업/img_10.png)

1. 팀원 등록 위해 팀원의 github 이메일을 입력
![](/images/34_Github 협업/img_11.png)

1. 이후 협업은 팀원의 할일과 동일하게 진행
### 팀원일 경우

**팀원의 할일**

- 메일로 들어가 초대된 github 승인
- VSCode에 Extensions에서 GitHub Pull Requests 설치
![](/images/34_Github 협업/img_12.png)

**협업을 위한 팀원의 기본 세팅**

1. 팀장 세팅이 완료 후 github repository 주소를 사용하여 Clone으로 받고 저장 (팀장 제외)
![](/images/34_Github 협업/img_13.png)

1. 프로젝트를 열었을 때 브랜치가 develop로 되어있는지 확인 (아닐 시 develop로 변경)
![](/images/34_Github 협업/img_14.png)

1. 새 브랜치 추가 브랜치 명은 자유롭게 작성(보통 개발 기능에 대한 것을 브랜치 명으로 함)
![](/images/34_Github 협업/img_15.png)

1. 개발 작업 시작 (Commit만 하고 Push는 절대 하지 않기)
**맡은 작업 완료 후 첫 번째로 PR 진행할 때**

1. 처음으로 하는 PR은 dev에 작성된게 없기 때문에 바로 PR 요청 작성
![](/images/34_Github 협업/img_16.png)

1. PR 요청 전 리뷰어 등록
![](/images/34_Github 협업/img_17.png)

1. 리뷰어 등록 후 PR을 승인할 사람 등록
![](/images/34_Github 협업/img_18.png)

1. Create를 눌러 PR 요청 (이렇게 하면 github에서 PR요청하는 걸 VSCode에서 한번에 진행)
![](/images/34_Github 협업/img_19.png)

1. PR 요청 후 등록된 리뷰어와 승인권자에게 PR요청 봐달라고 말하기
1. 승인이 거절되거나 수정해야 할 Conment가 달리면 해결하고 다시 처음부터 진행 
1. 승인이 완료되면 마지막으로 승인한 사람이 Merge를 진행하거나 본인이 Merge를 진행
**팀원이 PR 완료 후 PR 진행할 때**

1. PR를 진행하고 있는 팀원이 있는지 확인
1. PR를 진행하고 있는 팀원이 없고 전 PR이 완료되어 Merge된 상태면 PR 진행한다고 알리기
1. PR 진행 전 로컬 브랜치를 develop로 이동
![](/images/34_Github 협업/img_20.png)

1. Github에 있는 develop를 Pull로 당겨와 로컬에 있는 develop 브랜치를 최신화 시킴
![](/images/34_Github 협업/img_21.png)

1. 브랜치를 다시 작업하던 브랜치로 옮기고 로컬 develop에 병합 시도 (분기 → 병합)
![](/images/34_Github 협업/img_22.png)

![](/images/34_Github 협업/img_23.png)

1. 병합 시 충돌이 있는 파일들이 있는지 확인
  - Staged Changes는 충돌이 없는 파일들, Merge Changes는 충돌이 있는 파일들을 의미
1. 충돌이 있는 파일에 들어가서 수정
  - Incoming은 pull한 파일(즉, 로컬 develop 파일), Current는 내 파일(즉, 내 브랜치 파일)
  - 원하는 파일을 √로 선택하여 병합, 선택이 불가능할 경우 충돌이 나는 부분 하나씩 수정
![](/images/34_Github 협업/img_24.png)

![](/images/34_Github 협업/img_25.png)

1. Merge Changes에 파일이 충돌나는 파일이 없는지 확인하고 Continue를 눌러 병합 진행
![](/images/34_Github 협업/img_26.png)

1. PR 요청 작성
![](/images/34_Github 협업/img_27.png)

1. PR 요청 전 리뷰어 등록
![](/images/34_Github 협업/img_28.png)

1. 리뷰어 등록 후 PR을 승인할 사람 등록
![](/images/34_Github 협업/img_29.png)

1. Create를 눌러 PR 요청 (이렇게 하면 github에서 PR요청하는 걸 VSCode에서 한번에 진행)
![](/images/34_Github 협업/img_30.png)

1. PR 요청 후 등록된 리뷰어와 승인권자에게 PR요청 봐달라고 말하기
1. 승인이 거절되거나 수정해야 할 Conment가 달리면 해결하고 다시 처음부터 진행 
1. 승인이 완료되면 마지막으로 승인한 사람이 Merge를 진행하거나 본인이 Merge를 진행
**팀원으로부터 PR 요청이 들어왔을 때**

1. Github에 들어가서 Pull requests에 들어가면 요청 온 PR 클릭
![](/images/34_Github 협업/img_31.png)

1. 요청 온 PR에서 Files changed를 눌러 변경된 파일에 오류가 있는지 등을 체크
![](/images/34_Github 협업/img_32.png)

1. 확인 후 Comment를 남기거나 내가 승인권자면 Approve(승인), Request changes(코드 수정 필요)중 하나를 선택하고 Submit review를 클릭하여 리뷰를 남김
![](/images/34_Github 협업/img_33.png)

1. 마지막으로 승인한 팀원은 Merge Pull request를 눌러 병합 진행
![](/images/34_Github 협업/img_34.png)

**PR 요청과 Merge까지 완료 후 수정 사항이 생겼을 때**

- develop 브랜치로 변경 후 새 브랜치를 만들어 코드 수정 및 커밋하고 다시 PR부터 진행
