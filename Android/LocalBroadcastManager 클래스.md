# LocalBroadcastManager 클래스
* `Context`의 `sendBroadcast()` 메소드는 `Binder IPC`를 통해 `ActivityManagerService`를 거쳐야 하므로 속도에서 이점이 크지 않음.
* 또한 `Intent` 액션을 안다면 원치 않는 곳에서도 이벤트를 받아서 예기치 않는 일을 할 가능성도 있음.
* `sendBroadcast()`에서는 파라미터로 전달되는 `Intent`의  `setPackage()` 메소드를 사용해서 원하는 패키지에만 브로드캐스트를 전달할 수 있음.
* 다만 이 방식은 ICS(Android 4.0) 이상부터 동작함.
* 프로세스 간에 브로드캐스트를 보낼 필요가 없다면 `support-v4`에 포함된 `LocalBroadcastManager`의 사용을 고려해볼만 함.
* `LocalBroadcastManager`는 로컬 프로세스에서만 이벤트를 주고받을 수 있음.
* 시스템 브로드캐스트와는 구조가 완전히 다른데, `ActivityManagerService`를 거치지 않고 싱글톤인 `LocalBroadcastManager`에서 `registerReceiver()` 와 `sendBroadcast()`를 실행함.

```java
LocalBroadcastManager.getInstance(this).sendBroadcast(CalednarIntent.CHANGE_TIME);
...
LcalBroadcastManager.getInstance(this).registerReceivedr(..);
```

## LocalBroadcastManager의 장점
* 브로드캐스트하는 데이터가 다른 앱에서 캐치되지 않아서 안전함.
* 글로벌 브로드캐스트보다 속도가 빠름.

## LocalBroadcastManager 내부에서 Handler 사용
* `LocalbroadcastManager`에 등록된 `BroadcastReceiver`의 `sendBroadcast()` 메소드는 `BroadcastReceiver`를 바로 실행하지 않음.
* `Handler`에 `Message`를 보내서 메인 스레드에서 가능한 시점에 처리함.
* 따라서 메인 `Looper`의 `MessageQueue`에 쌓여 있는 게 많다면 처리가 늦어질 수도 있음.

## sendBroadcastSync() 메소드
* 그런데 인증 에러 같은 것을 `MessageQueue`에서 처리하면 문제가 될 수도 있음.
* `BroadcastReceiver`에서 바로 처리해야 하는데, 그렇지 않고 다른 작업을 먼저 한다면 타이밍상 엉뚱한 결과를 만들어 내는 경우가 생김
* 이때 쓰는 것이 `sendBroadcastSync()` 메소드임.
* 등록된 `BroadcastReceiver`는 동일한 스레드에서 실행됨.

## 앱 위젯에는 로컬 브로드캐스트가 전달되지 않음.
* `LocalBroadcastManager`에서 `sendBroadcast()`를 호출할 때 브로드캐스트 리시버의 한 종류인 앱 위젯에는 이벤트가 전달되지 않음.
* 앱 위젯은 홈 스크린에 설치되어 별도 프로세스에 있으므로, 글로벌 브로드캐스트를 사용해서 이벤트를 전달해야만 함.
* 다만 앱 위젯의 `onReceive()` 실행 위치는 다시 앱의 프로세스임.