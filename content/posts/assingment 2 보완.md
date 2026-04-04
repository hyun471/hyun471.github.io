---
title: "assingment 2 보완"
status: "published"
date: "2026-04-04"
type: "posts"
---

# 📚 Console Battle RPG

## 새롭게 알게 된 정보

**File('상대경로/.txt') : 경로 내에 .txt 파일 불러오기**

- file.readAsLinesSync() : 변수 file에 들어있는 값 한 줄 String으로 넣기
- file.writeAsStringSync() : ()안에 내용을 .txt에 덮어쓰기
```dart
//character.txt안에 50,10,5 라고 적혀있을 떄

import 'dart:io';

void main() {
  final file = File('data/character.txt');         // 파일 불러와 변수에 담기
  final lines = file.readAsLinesSync();            // 파일 내용 한줄의 문자열로 넣기
  final result1 = file.writeAsStringSync("승리");   // 파일에 () 내용으로 덮어쓰기
  final result2 = resultFile.writeAsStringSync(     // 파일 마지막에 () 내용 추가
  "승리",
   mode: FileMode.append,
);
  
  print(lines); // 한 줄씩 출력 50,10,5
}
```

- 절대 경로 : 최상위 경로부터 다 적는거 ⇒ 환경이 바뀌면 경로를 바꿔줘야함 = 잘 사용X
  예시 : File(’/Users/mk/Desktop/dart_morning_coding/monster.txt’)

- 상대 경로 : 현재 실행중인곳 위치 기반으로 상대적인 경로, 제일 앞에 / 없음
  예시 : final file = File(’monster.txt’)

**split(’, ’) : 문자열을 → 리스트로 나눔 (()괄호 안에 표식을 기준으로 구분하여 나눔)**

```dart
String data = "50,10,5";
List<String> parts = data.split(", ");
print(parts); // ['50', '10', '5'] String 리스트
```

**stdout.write() : 출력 창 입력 프로프트와 입력이 같은 줄에 나오게 함**

```dart
import 'dart:io';

void main() {
  stdout.write("캐릭터의 이름을 입력하세요: ");
  String name = stdin.readLineSync()!;
  print("입력한 이름은 $name 입니다.");
}
// 캐릭터의 이름을 입력하세요: dev
// 입력한 이름은 dev 입니다.
```

**Random() : dart:math에 들어 있는 클래스, 숫자 등을 무작위로 생성할 수 있음**

- Random().nextInt() : 0부터 ()안에 숫자까지 무작위로 하나 뽑는다.
```dart
import 'dart:math';

void main() {
  int max = 12;
  int randomNumber = Random().nextInt(max) + 1;
  print("1부터 $max 사이의 랜덤 숫자: $randomNumber");
} // 1부터 12까지 무작위 숫자
```

## Console Battle RPG

### 과제 시나리오

- 목표 : Battle RPG 게임 시스템 만들고 아래 기능 구현하기
**기능 요구 사항  : 다음 사항을 사용하여 만들기**

1.  플레이어

- Character 클래스 (속성 : 이름, 체력, 공격력, 방어력)
- 플레이어 스탯은 characters.txt 파일에서 읽어옴
- 플레이어 이름은 입력값을 사용
- 공격 : 몬스터 체력 - 플레이어 공격력
- 방어 : 플레이어 방어력 만큼 체력 회복 (체력 회복은 최대 체력을 넘을 수 없음)
2. 몬스터

- 몬스터 종류 및 스탯은 monsters.txt 파일에서 읽어옴 (속성: 이름, 체력, 공격력)
- 공격 : 플레이어 체력 + 플레이어 방어력 - 몬스터 공격력
3. 배틀

- 몬스터 출현
- 플레이어 턴부터 배틀시작, 만약 몬스터의 체력이 0이면 배틀 종료
- 플레이어 턴이 끝나면 몬스터 턴 시작 → 몬스터 공격
- 만약 플레이어 체력이 0이되면 배틀 종료
- 플레이어 또는 몬스터의 체력이 0이 될 때까지 반복
4. 배틀이 끝나거나 플레이어 사망 시 저장 여부

- 몬스터를 쓰러뜨렸을 경우 배틀을 계속 진행 여부 확인 → 계속 진행이면 다른 몬스터로 배틀 시작
- 몬스터를 쓰러뜨리거나 플레이어 사망 시 저장 여부 확인
- 저장 선택 시 → 결과 저장 → 프로그램 종료
- 저장 안 할 시 → 프로그램 종료
**필수 조건 : **

- 몬스터 및 플레이어 스탯은 .txt 파일로 불러와야 함
- 몬스터의 공격력은 무작위로 결정됨
(플레이어 방어력 < 몬스터 공격력 < 몬스터 스탯의 공격력이 최대 값)
- 몬스터 리스트에서 무작위의 몬스터 출현
- 배틀 시 또는 종료 시 남은 체력은 유지 되어야 함
- 저장 시 result.txt 파일에 결과(이름 - 남은 체력, 잡은 몬스터 수) 저장
**도전 조건:**

- 게임 시작 시 30%확률로 플레이어의 최대체력를 10 증가하게 만들기
- 배틀 도중 한번만 사용이 가능한 특수 아이템 만들기
- 몬스터는 3턴마다 방어력이 2씩 증가하게 만들기
- 예시 출력
![](/images/23_assingment 2 보완/img_01.png)

![](/images/23_assingment 2 보완/img_02.png)

## 구현 완료

- 구현 완료 
## 트러블슈팅 과정

**07-01 트러블 슈팅**

### int.parse()에서 FormatException 발생

- 문제 : int.parse(part[0]); 실행 시 에러가 발생함
```dart
final part = line.split(", ");
  int hp = int.parse(part[0]);
  
  FormatException: Invalid radix-10 number // 에러 발생
```

- 원인 : split(", ");은 쉼표 공백을 기준으로 나눠야 하는데 characters.txt에는 “50,10,5”의 데이터가 쉼표 뒤에 공백이 없는 형식이라 데이터를 나누지 못하고 “50,10,5”로 반환함
- 해결 : split(","); 처럼 공백 없는 구분자 사용으로 정리 또는 .trim()으로 각 값의 공백 제거
### PlayerModel() 생성자 호출 시 에러

- 문제 : main()함수에서 PlayerModel character = PlayerModel(); 생성자 호출 시 에러 발생
- 원인 : PlayerModel생성자는 PlayerModel(String inputName, int hp, int power, int shield)을 필수로 받도록 설계되어 있어 인자를 안 넣었기 때문에 오류 발생
- 해결 : 생성자 호출 시 모든 필수 값을 넘겨줌
### 클래스 안 생성자에서 텍스트 파일을 읽고 플레이어 스탯을 초기화

- 문제 : 클래스 안에서 직접 파일을 읽는 코드는, 인스턴스 생성보다 먼저 실행되기 때문에 문제가 발생
- 원인 : 클래스 인스턴스를 만들기도 전에 파일을 읽으면 실행을 허용하지 않음
  - 클래스 안에서 멤버 필드 선언과 동시에 파일을 읽으려고 하면 Dart에서는 필드 초기화 시점에서 동기적인 I/O 작업을 실행하는 걸 허용하지 않음
- 해결 : 파일을 읽는 것을 클래스 만들기 전에 만들기, 메서드 안에서 처리
  - 생성자가 아닌 별도의 메서드에서 파일을 읽고 값을 추출해서 클래스를 만들어야함
### 널 안정성(Null Safety) 관련 에러

- 문제 : 플레이어 값에 불러오는 파일이나, 입력값이 null 일수 도 있어 에러가 발생함
- 원인 : 입력값이 String?으로 null이 될 수 도 있음을 선언하였기 때문에 이후 항상 null 체크가 필요함
- 해결 : Dart는 Null Safety를 지원해서, 변수에 null이 들어갈 수  있는 겨우 명시적으로 ?를 붙여야함
  - null이 될 수 있는 변수는 ?를 붙여서 선언
  - 사용할 때는 null 여부를 체크하고 종료하는 방어 코드를 사용하여 !를 강제 사용
```dart
if (player == null) return;
player!.hp -= 10; // 확신이 있을 때만 !
```

  - 또는 사용할 때 안전하게 ?.를 사용해서 호출 
```dart
player?.show(); // player가 null이 아니면 호출
```

**07-04 트러블 슈팅**

### battle_rpg.dart 파일에 캐릭터, 몬스터 속성을 제외한 나머지가 한 코드에 작성

- 문제 : 현재 코드가 battle_rpg.dart에 몰려있어 메서드를 만들어 나눠주는 작업이 필요
- 원인 : 현재 코드에서도 작동은 잘하지만 관리가 어려워 동일한 기능들을 나눠주는 작업이 필요
- 해결 : 추상클래스로 캐릭터, 몬스터 상속해주고, 같은 기능인 공격, 방어를 메서드로 만들어줌
### selectedMonster 변수 에러

- 문제 : selectedMonster가 battleStart()에 전달되지 않음
- 원인 : selectedMonster를 선언하지 않고 rpg.battleStart(selectedMonster);에서 사용
- 해결 : monsterAppear()메서드에서 selectedMonster를 반환한 값을 main()에서 변수에 저장
### 배틀 시작시 플레이어 이름이 Player로 고정됨

- 문제 : main()에서 PlayerModel.playerStatsLoad(”player”)로 고정 호출해서 이름이 고정됨
- 원인 : main()과 gameStart()에서 PlayerModel을 각각 생성하여 객체가 중복됨
- 해결 : gameStart()에서 입력값을 받아서 PlayerModel 생성 후 BattleRPG 생성자에 전달하여 
           실제 플레이어 객체에 반영
### 에러 발생으로 인한 예외 처리

- 문제 : RangeError: max must be positive의 에러 발생
- 원인 : 몬스터의 리스트에서 랜덤으로 호출 시 리스트가 비어 있어 에러 발생
- 해결 : if문으로 monsterList가 비어있는 경우 몬스터 없음 출력과 결과 저장으로 예외 처리
### 죽은 몬스터가 다시 등장

- 문제 : 몬스터를 처치한 후 다시 몬스터 리스트에서 랜덤으로 나올 때 죽은 몬스터도 등장함
- 원인 : hp가 0인 몬스터가 몬스터 리스트에 남아있음
- 해결 : 몬스터 리스트에서 where를 사용하여 hp>0인 몬스터만 다시 뽑아서 List화 하여 사용