# ANR
> Application Not Responding (애플리케이션이 응답하지 않습니다.)

* 어느 동작에서 메인 스레드를 오랫동안 점유하고 있다는 의미
* 이 메시지를 통해 메인스레드 점유가 끝날 때 까지 대기할 것인지 프로세스를 종료할 것인지 사용자에게 묻는 과정을 거침
* 아무리 앱을 잘 만들어도 단말기의 상태가 좋지 않으면 ANR이 발생할 수 있기 때문에 완벽하게 피할 수는 없음.
* 따라서 ANR이 발생할 수 있는 케이스를 최대한 줄이는 것이 그나마 가능한 목표.


## ANR 타임아웃

** 젤리빈 이상**

```java
// How long we allow a receiver to run before giving up on it.
static final int BROADCAST_FG_TIMEOUT = 10 * 1000;
static final int BROADCAST_BG_TIMEOUT = 60 * 1000;

// How long we wait until we timeout on key dispatching.
static final int KEY_DISPATCHING_TIMEOUT = 5 * 1000;
```

* InputDispatching(키 이벤트와 터치 이벤트를 포함) 타임아웃은 ActivityManagerService 뿐만 아니라, 네이티브 코드에도 동일한 값이 적용되어 있음.

**ICS 이하**
```java
// How long we allow a receiver to run befoe giving up on it.
static final int BROADCAST_TIMEOUT = 10 * 1000;

// How long we wait for a service to finish executing.
static final int SERVICE_TIMEOUT = 20 * 1000;

// How long we wait until we timeout on key dispatching.
static final int KEY_DISPATCHING_TIMEOUT = 5 * 1000;
```

* 젤리빈부터 생긴 차이
  * `SERVICE_TIMEOUT`이 보이지 않음. 클래스가 분리되면서 `com.android.server.am.ActiveServices`로 위치가 변경됨.
  * 브로드캐스트 리시버 타임아웃이 포그라운드와 백그라운드 2단계로 바뀜. 
  * 기존에는 10초만 지나면 ANR이 발생하였지만 이제는 특별히 명시하지 않으면 1분이면 타임아웃이 됨.
  * 포그라운드로 명시하는 방법은 sendBroadcast()에 전달하는 인텐트에 `Intent.FLAG_RECEIVER_FOREGROUND`를 추가흔느 것.
  * `ActivityManagerService`에는 포그라운드와 백그라운드 용도의 `BroadcastQueue`가 각각 있는데 큐에 쌓인 순서에 관계 없이 포그라운드 용도의 `BroadcastQueue`를 먼저 처리함.

## 프레임워크에서 ANR 판단

* ANR 발생 시에는 `ActivityManagerService`의 `appNotResponding()` 메소드에서 다이얼로그를 띄우는 등의 일을 처리함.

### 브로드캐스트 리시버와 서비스는 Handler를 이용해서 ANR 판단

* 브로드캐스트 리시버와 서비스는 시작 전에 `Handler`의 `sendMessageAtTime()` 메소드를 사용해서 Message를 보내고 타임아웃이 되면 `appNotResponding()` 메소드를 호출
* 타임 아웃 내에 실행이 끝나면 Message를 제거함.
* `ActivityManagerService`는 `system_server`로 떠있는 별도의 프로세스에서 동작하기 때문에 이 Handler는 앱 프로세스의 메인 스레드와는 관련이 없음.

### 화면 터치와 키 입력 시 ANR 발생
* `InputDispatching` 타임아웃은 네이티브 소스인 `/frameworks/base/services/input/InputDispatcher.cpp`에 지정되어있음.
```c++
const nsec_t DEFAULT_INPUT_DISPATCHING_TIMEOUT = 5000 * 1000000LL; // 5 sec
```

### 화면 터치와 키 입력 전달 메커니즘.
* 커널에서 네이티브 단을 거쳐서 앱에 입력 이벤트가 전달되는 것으로, `InputReader`에서 `EventHub`를 통해 커널에서 이벤트를 가져오고 `InputDispatcher`는 이벤트를 전달함.
* `Activity`는 `Window`를 갖고 `Window`는 `ViewRootImpl`과 일대일로 매핑
* `ViewRootImpl` 내부 클래스인 `WindowInputEventReceiver`에서 이벤트를 전달받아서 하위 ViewGroup / View로 전달
* `WindowInputEventReceiver`에서 전달받는 파라미터는 `InputEvent`로 `MotionEvent`와 `KeyEvent`의 상위 추상 클래스

### 화면 터치와 키 입력에서 ANR 발생
* `InputDispatcher.cpp`에서 주요 메소드는 `dispatchMotionLocked()`와 `dispatchKeyLocked()`로, 각각 터치이벤트 키 이벤트를 전달함.
* `findFocusedWindowTargetsLocked()` 메소드에서 이벤트를 전달할 Window를 먼저 찾는데, `isWindowReadyForMoreInputLocked()` 메소드에서 기존 이벤트를 처리하느라 대기해야 하는지를 판단함.
* 이벤트를 전달핮비 않고 기다리다가 타임아웃이 되면 `onANRLocked()` 메소드를 호출하고, `notifyANR()` 메소드를 거쳐서 `ActivityManagerService`의 `appNotResponding()` 메소드에 이르게 됨.
* 키 이벤트
  * 메인 스레드를 어디선가 이미 점유하고 있다면 키 이벤트를 전달받지 못함.
  * 키가 눌리고 5초 이상 지연시 ANR 발생 
* 터치 이벤트
  * 메인스레드가 어디선가 사용중이라면 대기
  * 타임아웃이 된다고 바로 ANR이 발생하지는 않음.
  * 이어서 터치 이벤트가  왔을 때 두 번째 터치이벤트가 전달되지 않는 시간이 타임아웃되면 ANR이 발생

### Message 처리 각각이 5초 이내라도 총합 처리 시간 영향 있음.
특정 Message의 처리가 5초가 넘더라도 그 사이에 터치가 없을 때에는 문제가 발생하지 않음.

```java
private Handler handler = new Handler() {      
    public void handleMessage(Message msg) {    // handleMessage()  메소드를 보면 Message당 2초당 소요
        Log.d(TAG, "handleMessage");
        SystemClock.sleep(2000);
    };
};

private void onClickSendMessages(View v) {
    for(int i = 0; i < 5; i++) {
        handler.sendEmptyMessage(0);         // onClickSendMessage() 메소드에서 5개의 메시지를 연속으로 보냄.
    }
}
```

* 따라서 5개의 message를 처리하는 시간은 총 10초
* 가만히 두면 문제가 없지만, `onClickSendMessage()`를 실행한 후 곧바로 화면을 두 번 이상 터치하면  ANR이 발생.
* 앞에 쌓여있는 Message를 먼저 처리하느라 터치 이벤트에 대한 처리가 지연됨.


### 서비스나 브로드캐스트 리시버에서도 5초 이내로 메시지 처리가 필요
*  onReceive() 실행이 다 끝난 다음에야 터치 이벤트가 전달되는데 onReceive가 5초 이상 실행중이라면 ANR이 발생
*  브로드캐스트 리시버나 서비스도 액티비티가 떠있는 상태를 고려해서, 타임아웃을 5초라고 생각하는 편이 나음.
*  결론적으로 브로드캐스트 리서버의 경우에는 오래 걸리는 작업이 있다면 서비스로 넘겨서 실행하고, 서비스에서는 다시 백그라운드 스레드를 이용해야 함.

