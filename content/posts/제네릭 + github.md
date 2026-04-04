---
title: "제네릭 + github"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 제네릭

- 특정 타입에 의존하지 않고, 여러 타입에 대한 동일한 코드를 적용할 수 있어 재사용성이 높다.
## 제네릭 함수

- 함수가 입력과 출력 타입을 일반화해서, 호출할 때 타입을 정할 수 있게 만든다.
- 필수 구성 요소 : T + 함수<T>(T 매개변수) {코드} 
  - 여기서 T는 임의의 타입 파라미터임
  - 타입 파라미터는 보통 영어 대문자 한 글자(E, T, S, K, V 등)로 표현
```dart
T echo<T>(T value) {          // T타입 (T타입은 아직 정해지지 않는 타입)
  return value;
}

void main() {
  print(echo<int>(123));      // 123 (int 타입)
  print(echo<String>('hi'));  // hi (String 타입)
}
```

## 제네릭 클래스

- 클래스 안에서 사용할 타입을 정하지 않고, 나중에 인스턴스를 만들 때 정한다.
- 필수 구성 요소 : class + 클래스<T> {T 멤버변수 + 생성자 + 메서드}
  - 여기서 T는 임의의 타입 파라미터임
  - 타입 파라미터는 보통 영어 대문자 한 글자(E, T, S, K, V 등)로 표현
```dart
class Box<T> {
  T value;           // T 멤버변수
  Box(this.value);   // 생성자

  void show() {      // 메서드
    print(value);
  }
}

void main() {
  var intBox = Box<int>(123);          // Box<int> 타입 인스턴스 생성
  intBox.show();                       // 출력: 123

  var strBox = Box<String>('hello');   // Box<String> 타입 인스턴스 생성
  strBox.show();                       // 출력: hello
}
```

# 💡 세미나 학습

## GitHub

### GitHub 특징

- Git는 Git을 기반으로 한 클라우드 코드 저장 서비스이다. 
- 코드 작업을 위한 협업 기능, 버전 관리 등이 가능하여 현재 IT에서 가장 많이 사용한다.
### GitHub 사용법

- 모든 명령어는 터미널에서 사용한다.
1. Git 초기화 (명령어 : git init)
  - 로컬(내 컴퓨터) 프로젝트 폴더를 Git 저장소로 초기화함
  - ‘.git’ 폴더가 생성되며, 이 안에 스테이징 영역(staging area)과 로컬 프랜치(main) 정보가 관리됨
  - Git은 파일 변경 사항을 추적 할 수 있는 상태가 됨
1. 스테이징 영역 (명령어 : git add)
  - 스테이징 영역은 내 pc의 현재 브랜치와 비교해서 변경된 파일을 추적하는 중간 저장소 역할임
  - ‘git add <파일명>’ : 해당 파일만 스테이징 영역에 추가함
  - ‘git add . ‘ : 현재 프로젝트 폴더 내의 모든 변경 파일을 스테이징 영역에 추가함 
1. commit : (명령어 : git commit -m “변경내용”)
  - 스테이징 영역에 있는 변경사항을 로컬 브랜치에 저장하는 명령어
  - -m “변경내용” 를 붙여야 commit에 대한 설명과 함께 저장됨
  - ‘git commit’ 은 GitHub에 업로드 하지 않고 내 컴퓨터에만 저장
  - 협업에서 변경 내용 작성 시 의도를 전달하기 위하여 구조적 요소가 포함되어 있음
    - fix : 코드에서 버그를 수정했을 때 변경 내용 앞에 붙임 (ex: “fix : 로그인 오류 수정”)
    - feat : 코드에 새 기능이 추가되었을 때 (ex: “feat : 로그인 기능 추가”)
  - 이외에 자세한 기능은 아래 주소를 참고
1. 원격 저장소 연결 (명령어 : git remote add)
  - GitHub에 만든 저장소와 내 로컬 Git 프로젝트를 연결함
  - 기본적으로 다음과 같이 사용 : git remote add origin https://github.com/사용자명/저장소명.git
  - ‘git remote -v’를 사용하여 저장소 목록을 확인할 수 있음
1. GitHub로 업로드 (명령어 : git push)
  - 내 로컬 브랜치의 내용을 GitHub의 브랜치에 업로드함