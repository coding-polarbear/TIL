# Messenger 클래스
> 바인더 콜백을 내부적으로 래핑해서 바운드 서비스와 클라이언트 간에 Handler로 메시지를 보내고 처리하는 방식을 제공.

## Messenger 클래스의 기본 내용
* `Parcelable` 인터페이스를 구현해서 프로세스 간에 전달할 수 있는 객체
* 2개의 생성자가 존재
  * `Messenger(Handler target)` : Handler를 감싼 것으로 클라이언트와 바운드 서비스 양쪽에 있음.
  * `Messenger(IBinder target)` : Binder Proxy를 생성하는 것으로 클라이언트에 있음.
* `aidl`을 내부적으로 사용하는데 `IMessenger` 인터페이스로 되어 있음.
  * Handler의 내부 클래스인 `MessengerImpl`에서 `IMessenger.Stub`를 구현함.
* `Handler`에 `Message`를 보낼 때 `replyTo`에 값을 되돌려줄 `Messenger`로 지정할 수 있음.
* `Messenger`의 `send()` 메소드는 결과적으로 바인더 통신을 통해 `Stub` 메소드를 호출함.
  * 리모트 통신이기 때문에 `send()` 메소드는 `RemoteException`을 던질 수 있음.