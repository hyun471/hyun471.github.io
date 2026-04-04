---
title: "MVVM 구조"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 MVVM 구조

## MVVM이란

- Model +View + ViewModel의 약자
- Model : 데이터를 다루는 곳을 의미하며 주로 데이터의 구조와 처리 로직을 담당
- View : 사용자에게 보이는 것을 의미하며 주로 UI화면 구성을 담당
- ViewModel : 상태를 관리하는 것을 의미하며 주로 View와 Model 사이의 상태 관리를 담당
### MVVM 디렉토리 구조

- 디렉토리 구조에 역할 기반과 기능 기반 중 기능 기반이 트렌드 (기능 기반 디렉토리 정리 링크)
- 따라서 기능 기반 디렉토리에 MVVM 디렉토리를 추가하여 다루는 것이 중요
- 기능 기반 MVVM 디렉토리 구조
  - 기능 기반에 따라 디렉토리 분리 
  - 앱 전체에서 공통으로 작업되는 부분은 공통 폴더에 분리
```dart
// 필요한 폴더, 파일만 사용 또는 생성
📁 lib/
├── main.dart                   // 앱 시작점
├── theme.dart                  // 공통 테마 정의
├── 📁 commons/                // 공통 UI, 데이터 모델 등
│   ├── 📁 views/              // 공통 UI 정의
│   │   └── button.dart
│   ├── 📁 models/             // 공통 데이터 모델, 데이터 로직 등
│   │   └── profile_model.dart
│   └── 📁 repository/          // 공통 서비스(API 통신, 유저인증, 저장 등)
│       └── profile.dart
└── 📁 pages/                   // 각 화면 별 구성될 페이지들
    ├── 📁 home/                // Home 화면에 사용될 것들
    │   ├── 📁 views/           // Home 화면에만 사용될 UI 위젯들
    │   ├── 📁 models/          // Home 화면에만 사용될 데이터
    │   ├── 📁 viewmodels/      // Home 화면에만 사용될 상태 관리
    │   ├── 📁 repository/      // Home 화면에만 사용될 서비스 로직
    │   └── home_page.dart       // HOme 화면 페이지 구성 로직
    ├── 📁 result/
    │   ├── 📁 views/            // result 화면에만 사용될 UI 위젯들
    │   ├── 📁 models/           // result 화면에만 사용될 데이터
    │   ├── 📁 viewmodels/       // result 화면에만 사용될 상태 관리
    │   ├── 📁 repository/         // result 화면에만 사용될 서비스 로직
    │   └── result_page.dart     // result 화면 페이지 구성 로직

```
