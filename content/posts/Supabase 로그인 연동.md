---
title: "Supabase 로그인 연동"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 Supabase 로그인 연동 (google, apple)

## Supabase Google 로그인 연동

**Supabase에 Google 로그인 연동**

- Supabase에 구글 로그인 연동을 하기 위하여 Supabase 프로젝트 → Authentication → Sign in / Providers에서 Auth Providers의 Google을 Enabled로 해야함
- 생성된 Google Cloud OAuth Web 클라이언트 ID와 클라이언트 Secret 입력
(ID 및 Secret은 밑에 있는 Google Cloud OAuth 클라이언트 ID 생성에서 확인)
- Supabase에서 제공하는 Callback URL은 google Cloud에서 사용
**Google Cloud OAuth 클라이언트 ID 생성**

- Google Cloud Console 해당 프로젝트(없으면 생성) → 왼쪽 바에서 API 및 서비스 → 사용자 인증 정보 → 사용자 인증 정보 만들기 → OAuth 2.0 클라이언트 ID
- Web Client ID → 여기에 Callback URL 넣기
(Supabase는 Web Client ID 기반으로 OAuth 요청을 시작하기 때문에 Web OAuth Client ID 필요)
  - 승인된 리디렉션 URL은 Supabase에서 제공하는 Callback URL 입력 
(밑에 있는 Supabas Google 로그인 연동 부분 확인하여 가져오기)
  - Client Secret 잘 보관
![](/images/33_Supabase 로그인 연동/img_01.png)

- Android Client ID → Android 폰에서 로그인 시 필요
(테스트의 경우 테스터들의 디버그 디지털 지문을 넣어서 사용, 배포 시 배포 용 디지털 지문 사용)
- IOS Client ID → IOS 폰에서 로그인 시 필요
(앱 번들 ID 입력)
**구글 로그인 테스트를 위한 테스터 이메일 등록**

- 왼쪽 바에서 API 및 서비스 → OAuth 동의 화면 → 대상 → 테스트 사용자 등록
**VScode에서 로그인 구현**

- android → app → src → main → AndroidManifest.xml에서 설정 (Android폰에서 로그인)
```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data android:scheme="dailymoji" android:host="login-callback"/>
</intent-filter>
```

  - <action android:name="android.intent.action.VIEW"/> :
(이 Activity가 외부에서 호출되는 URL(Intent)도 처리 가능함을 의미)
  - <category android:name="android.intent.category.DEFAULT"/> :
(기본 카테고리, 필수)
  - <category android:name="android.intent.category.BROWSABLE"/> :
(브라우저/외부 앱에서 호출 가능하도록 허용)
  - <data android:scheme="dailymoji" android:host="login-callback"/> :
(Flutter 코드에서 지정한 redirectTo URL과 일치해야 앱으로 돌아올 수 있음)
- ios → Runner→ Info.plist에서 설정 (IOS폰에서 로그인) 
```xml
	<key>CFBundleURLTypes</key>
	<array>
		<dict>
			<key>CFBundleURLSchemes</key>
			<array>
				<string>dailymoji</string>
			</array>
		</dict>
	</array>
```

  - CFBundleURLTypes : URL Scheme을 등록하는 키
  - CFBundleURLSchemes : 앱이 처리할 커스텀 스킴 (ios는 스킴만 일치하면 콜백 됨)
- Supabase의 OAuth를 이용하여 구글 로그인 호출
```dart
await Supabase.instance.client.auth.signInWithOAuth(
        OAuthProvider.google,
        authScreenLaunchMode: LaunchMode.externalApplication,
        redirectTo: 'dailymoji://login-callback',
      );
```

  - signInWithOAuth의 OAuthProvider로 구글을 입력
  - redirectTo: 콜백할 주소 입력 (흔히 커스텀 스킴://호스트 형식 사용)
    - Android는 AndroidManifest.xml에서 <intent-filter>에 등록된 주소가 동일해야함
AndroidManifest.xml의 스킴://호스트 == redirectTo:의 스킴://호스트
    - ios는 Info.plist에 등록된 CFBundleURLSchemes의 Value값에 등록된 스킴이 동일해야함
Info.plist의 스킴 == redirectTo:의 스킴
## Supabase Apple 로그인 연동

**Supabase에 Apple 로그인 연동**

- Supabase에 애플 로그인 연동을 하기 위하여 Supabase 프로젝트 → Authentication → Sign in / Providers에서 Auth Providers의 Apple을 Enabled로 해야함
- Apple Developer 계정에서 생성한 Service ID 입력 및 Secret Key에 JWT 파일 업로드
(Secret Key은 아래 Apple Developer 계정에서 OAuth 클라이언트 생성 부분 참고)
- Supabase에서 제공하는 Callback URL은 Apple Developer에서 사용
**Apple Developer 계정에서 OAuth 클라이언트 생성**

- Apple 로그인용 서비스 ID 생성 (Identifiers → Services IDs → + 새로 만들기)
  - Identifier : 서비스 ID (예: com.example.dailymoji.signin)
  - Enable “Sign in with Apple” 체크
  - Web domain, Return URLs 등록 (Supabase Dashboard → Callback URL 입력)
![](/images/33_Supabase 로그인 연동/img_02.png)

- Key 생성 (Certificates, Identifiers & Profiles → Keys → + 새로 만들기)
  - Sign in with Apple 선택
  - 생성 후 다운로드 → 파일 확장자가 .p8
- JWT 파일 만들기
  - Supabase에서 요구하는 Secret Key 형태는 Key ID, Team ID, Bundle ID가 있는 JWT 파일 형태
  - .p8 + Key ID + Team ID + Bundle ID 를 이용하여 Supabase가 요구하는 형식으로 JWT 인코딩
(ChatGPT 도움으로 Python 스크립트를 통해 인코딩 완료)
**VScode에서 로그인 구현**

- android → app → src → main → AndroidManifest.xml에서 설정 (Android폰에서 로그인)
```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data android:scheme="dailymoji" android:host="login-callback"/>
</intent-filter>
```

  - <action android:name="android.intent.action.VIEW"/> :
(이 Activity가 외부에서 호출되는 URL(Intent)도 처리 가능함을 의미)
  - <category android:name="android.intent.category.DEFAULT"/> :
(기본 카테고리, 필수)
  - <category android:name="android.intent.category.BROWSABLE"/> :
(브라우저/외부 앱에서 호출 가능하도록 허용)
  - <data android:scheme="dailymoji" android:host="login-callback"/> :
(Flutter 코드에서 지정한 redirectTo URL과 일치해야 앱으로 돌아올 수 있음)
- ios → Runner→ Info.plist에서 설정 (IOS폰에서 로그인) 
```xml
	<key>CFBundleURLTypes</key>
	<array>
		<dict>
			<key>CFBundleURLSchemes</key>
			<array>
				<string>dailymoji</string>
			</array>
		</dict>
	</array>
```

  - CFBundleURLTypes : URL Scheme을 등록하는 키
  - CFBundleURLSchemes : 앱이 처리할 커스텀 스킴 (ios는 스킴만 일치하면 콜백 됨)
- Supabase의 OAuth를 이용하여 구글 로그인 호출
```dart
await Supabase.instance.client.auth.signInWithOAuth(
        OAuthProvider.google,
        authScreenLaunchMode: LaunchMode.externalApplication,
        redirectTo: 'dailymoji://login-callback',
      );
```

  - signInWithOAuth의 OAuthProvider로 구글을 입력
  - redirectTo: 콜백할 주소 입력 (흔히 커스텀 스킴://호스트 형식 사용)
    - Android는 AndroidManifest.xml에서 <intent-filter>에 등록된 주소가 동일해야함
AndroidManifest.xml의 스킴://호스트 == redirectTo:의 스킴://호스트
    - ios는 Info.plist에 등록된 CFBundleURLSchemes의 Value값에 등록된 스킴이 동일해야함
Info.plist의 스킴 == redirectTo:의 스킴