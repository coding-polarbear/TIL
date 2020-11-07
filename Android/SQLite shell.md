# SQLite Shell

## 닷 커맨드
* 닷 커맨드라고 불리는 명령어 모음집이 존재함.
* 말 그대로 닷(.) 으로 시작하고 다른 명령어처럼 세미콜론(;)을 쓰지 않음.
* 이 모음에는 약 30여개의 명령어가 있는데, 주로 사용하는 것은 그렇게 많지 않음.
* 닷 커맨드 모음
  * `.tables` : 테이블 목록 보기
  * `.schema` :  스키마 확인
  * `.headers on` : 조회할 때 칼럼명 헤더를 보는 옵션

## 데이터베이스 명령어 실행
* `sqlite shell`에서 다양한 데이터베이스 명령어를 실행할 수 있음.
  * http://www.sqlite.org/lang.html
  * http://www.tutorialspoint.com/sqlite/

## PRAGMA 명령어
* DB의 환경 변수나 상태 플래그를 가져오거나 변경할 때 사용함.
* 안드로이드의 `SQLiteDatabase` 클래스에서는 메소드 개수가 많지 않음.
* PRAGMA 명령어를 써서 앱의 환경에 맞는 튜닝도 가능함.
* http://www.tutorialspoint.com/sqlite/sqlite_pragma.htm
