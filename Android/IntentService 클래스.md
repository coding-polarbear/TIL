# IntentService 클래스.md

* 서비스에서 멀티 스레딩이 필요한 경우가 많지는 않음.
* 동시에 여러 요청을 처리할 필요가 없으면 `IntentService`를 활용하는 것도 좋은 방법.
* `IntentService`는 내부적으로 1개의 백그라운드 스레드를 가지고 전달된 `Intent`를 순차적으로 처리함.
* IntentService에서는 백그라운드 스레드에서 실행되는 `onHandleIntent(Intent)` 메소드만 구현하면 됨.

```java
public class NewsReaderService extends IntentService {
    public NewsReaderService() {
        super("NewsReader");        // IntentService에는 기본 생성자가 없기 때문에 생성자를 같이 추가 해야 함.
    }

    @Override
    protected void onHandleIntent(Intent intent) {

    }
}
```

* IntentService에서 내부적으로 구현한 `onStartCommand()` 메소드의 기본 리턴 값은 `START_NOT_STICKY`임.
* 이 값을 변경하려면 생성자에서 `setIntentRedelivery(true)`를 호출해야함.
* `onCreate()`에서 `HandlerThread`를 생성하고 시작하면서 `HandlerThread`의 `Looper`와 연결된 `Handler`를 만듬.
* `onStartCommand()` 메소드는 실행도리 때마다 `Handler`에 메시지를 보내고 `handleMessage()` 메소드에서는 `onHandleIntent()`가 끝나면 바로 `stopSelf(int startId)`를 호출해서 서비스를 종료함.

##  IntentService에서 Toast 띄우는 문제
* Toast.show()를 실행하면 바인더 통신을 통해 `system_server` 프로세스에서 `NotificationManagerService`의 `enqueToast()` 메소드를 호출함.
* 이때 파라미터로 바인더 콜백이 전달되고, 콜백에서는 2가지 작업을 수행함.
  * 화면에 Toast를 보여주는 작업
  * 일정 시간 이후에 제거하는 작업
* `IntentService`에서는 `onHandleIntent()`를 실행한 이후에 바로 `stopSelf()`를 호출하고, `onDestroy()`에서는  `HandlerThread`에서 생성한 `Looper`를 종료하는 `Looper.quit()`를 호출함.
* Looper가 종료되면서 생기는 현상은 `Looper`의 `MessageQueue`에 전달되는 콜백이 실행되지 않는 다는 점.
* Looper가 종료되면서 Toast를 보여주는 콜백은 실행되었지만 제거하는 콜백은 실행되지 않을 수도 있음.
* Toast는 가급적 메인 스레드에서 띄우는게 맞음
* 백그라운드 스레드에서 그냥 쓰면 크래시가 나기 때문에 결국 쓰지 않게 되지만, `IntentService`에서 써보니 잘 되네 하고 넘어가면 안됨
* `Looper.getMainLooper()`가 전달된 Handler에서 Toast를 띄우는 것이 더 적절함.

```java
new Handler(Looper.getMainLooper()).post(new Runnable() {
    @Override
    Toast.makeText(SomeActivity.thius, "Some Notice", Toast.LENGTH_LONG).show();
});
```