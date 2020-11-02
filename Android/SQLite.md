# SQLite
*  로컬DB이지만 속도가 그렇게 빠르지는 않음.
*  딱 혼자서 슬 수 있는 정도.
*  `SQLite`는 네이티브 라이브러리에 포함되어 있고 프레임웤를 거쳐서 접근하고 사용함.

## db 내용 확인
* `SQLite` db 파일은 `/data/data/패키지명/databases`에 저장됨.
* 단말에서는 일반적으로 db파일에 직접 접근하거나 쿼리를 실행할 수 없음 (루팅한 것이 아니라면)
* 개발 시에 에뮬레이터에서 디비를 확인하려면, 2가지 방법이 존재
  * sqlite shell에서 query를 실행
  * adb pull을 통해 db 파일을 가져와서, SQLite Database Browser 같은 툴로 데이터를 확인하고 쿼리를 실행

## db 파일 목록 조회
```
ls - $ /data/data/*/databases
```

* /data/data 아래에서 프로세스명 위체에 providers가 들어가는 것들을 보면 콘텐트 프로바이더에서 어떤 db를 사용하는지 볼 수 있음.
* 단말에 깔린 기본 앱 (캘린더, 주소록 등)이나 미디어 데이터, 시스템 설정 등도 컨텐트 프로바이더를 제공함.
* db 파일 목록을 보다 보면 `.db` 뿐만 아니라 `db-jounal`, `db-wal`, `db-shm` 등이 붙은 확장자 파일을 볼 수 있는데, SQLite에서 트랜잭션(`atomic commit and rollback) 구현 방식에 따른 것.
* 디폴트는 rollback-journal(-journal 파일 사용)이고 다른 옵션으로  `Write-Ahead Logging` 이 있음.

