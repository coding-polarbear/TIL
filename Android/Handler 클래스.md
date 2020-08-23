# Handler 클래스

> Handler 클래스는 Message를 MessageQueue에 넣는 기능과 MessageQueue에서 꺼내 처리하는 기능을 함께 제공함.

## Handler 생성자

* Handler()
* Handler(Handler.Callback callback)
* Handler(Looper looper)
* Handler(Looper looper, Handler.Callback callback)

여기서 Looper는 생성자를 호출하는 스레드의 Looper를 사용하겠다는 의미이며,

따라서 메인스레드에서 Hander 기본생성자는 ActivityThread에서 생성한 메인 Looper를 사용함.

### 백그라운드 스레드에서 Handler 기본생성자를 사용하려면 Looper 필요
* 백그라운드 스레드에서 Handler 기본 생성자를 사용하면, Looper가 준비되어 있지 않을때 `RuntimeException`이 발생함.
* 따라서 백그라운드 스레드에서 Handler 기본 생성자를 사용하기 위해서는 `Looper.prepare()`를 실행해서 Looper를 준비해야함.
* 내부적으로 `prepare()` 메소드는 MessageQueue를 생성하는 것 외에 별다른 동작을 하지 않음.

```java
public class LooperThread extends Thread {
    public Handler mHandler;

    public void run() {
        Looper.prepare();
        mHanlder = new Handler() {
            public void handleMessage(Message message) {
                // 여기서 메세지 처리
            }
        }
    }
}
```

* LooperThread에서 스레드를 시작하면 Looper.loop()에 무한반복문이 있기 때문에 해당 스레드는 종료되지 않음.
* mHandler에서 postXxx()나 sendXxx()를 사용하면 스레드 내에서 handleMessage를 실행함. 


### 호출 위치가 메인스레드인지 확인이 쉽지 않음

* 개발 중 Looper가 준비되지 않아서 `RuntimeException`을 만나는 경우가 종종 있음.
* 메시지 호출 스택이 깊어지면 호출 위칭가 메인 스레드인지 백그라운드 스레드인지 확인이 금방 안되기도 함.
* Hanlder를 생성할 때 현재 스레드와 다른 스레드에서 실행해야하는 상황이 있을 경우 3번째나 4번째 생성자를 사용하여 해당 스레드의 looper를 넣어주면 쉽게 해결 할 수 있음.

## Handler 동작
* `sendEmptyMessage()`, `sendEmptyMessageDelayed()`, `sendEmptyMessageAtTime()` 메소드는 `Message`의 `what` 값을 전달함.
* `~Delayed()`로 끝나는 메소드는 내부적으로 `~AtTime()` 메소드를 호출함.
* `sendMessageAtFrontOfQueue()` 나 `postAtFrontOfQueue()` 메소드는 특별한 상황이 아니면 쓰지 말라는 가이드가 있음. 

### dispatchMessage() 메소드

```java
public void dispatchMessage(Message msg) {
    if(msg.callback != null) { // callback Runnable이 있으면 실행하고 아니면 handleMessage()를 호출
        hanldeCallback(msg);
    } else {
        if(mCallback != null) {
            if(mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}

private static void handleCallback(Message message) {
    message.callback.run();
}
```

* `dispatchMessage()`는 public 메소드 이기 때문에, 드믈긴 하지만 직접 호출하는 경우도 있음.
* 이런 경우에는 MessageQueue를 거치지 않고 직접 Message를 처리함.

## Handler의 용도

### 백그라운드 스레드에서 UI 업데이트
> 백그라운드 스레드에서 네트워크나 DB 작업 등을 하다가 UI를 업데이트할 때 사용할 수 있음. `AsyncTask`에서도 내부적으로  Handler를 이용하여 `onPostExecute()` 메소드를 실행해서 UI를 업데이트함.

### 메인스레드에서 다음 작업 예약 
* UI 작업 중에 다음 UI 갱신 작업을 MessageQueue에 넣어서 예약.
* Handler에서 Message를 보내면 현재 작업이 끝난 이후의 다음 타이밍에 Message를 처리함.

### 반복 UI 갱신
> 반복해서 UI를 갱신

```java
private static fianl int DELAY_TIME = 2000;

private Runnable updateTimeTask = new Runnable() {
    @Override
    systemInfo.setText(monitorService.getSystemInfo());
    handler.postDelayed(this, DELAY_TIME);  // UI갱신이 끝나고 postDelayed() 에서 Runnable 자체를 전달하여 계속 반복
}
```

### 시간 제한
> 시간을 제한할 때 사용. 안드로이드 내부에서 ANR을 판단할 때도 사용하는 방법


ex) bluetooth Le 스캔 시간 제한


```java
private static final long SCAN_PERIOD = 10000;

private void scanLeDevice(final boolean enable) {
    if(enable) {
        mHandler.postDelayed(new Runnable() {   
            // postDelayed(Runnable) 메소드에서 stopLeScan()을 10초 후 실행하도록 Runnable Message를 전달함.
            @Override
            public void run() {
                mScanning = false;
                mBluetoothAdapter.stopLeScan(mLeScanCallback);
            }
        }, SCAN_PERIOD);
    }
    mBluetoothAdapter.startLeScan(mLeScanCallback);     // startLeScan 실행
}
```

* 앱에서 많이 사용하는 방법으로,  몇초 내에 백키를 반복해서 누를 때에만 앱이 종료되도록 할 때에도 같은 방식을 사용함.

## Handler의 타이밍 이슈
* 원하는 동작 시점과 실제 동작 시점에서 차이가 생기는 타이밍 이슈는 대부분 메인스레드와 Handler를 이용하면 단순하게 해결할 수 있음.
* MessageQueue에서 Message를 하나 꺼내오면 onCreate()에서 onResume() 까지 쭉 실행됨.
* 따라서 onCreate()에서 Handler의 post()에 전달된 Runnable은 onResume() 이후에 실행됨.

### 지연 Message는 처리 시점을 보장할 수 없음.

* Handler에서 `~Delayed()` 나 `~AtTime()` 메소드에 전달된 지연 Message는 지연 시간(delay time)을 정확하게 보장하지 않음.
* MessageQueue에서 먼저 꺼낸 Message의 처리가 오래 걸린다면 실행이 당연히 늦어짐.

```java
Handler handler = new Handler();
handler.postDelayed(new Runnable() {
    @Override
    public void run() {
        Log.d(TAG, "200 delay");
    }
}, 200);

handler.post(new Runnable() {
    @Override
    public void run() {
        Log.d(TAG, "just");
        System.clock.sleep(500);
    }
});
```

* 단일 스레드의 규칙때문에 앞의 작업을 끝내야만 뒤의 작업을 처리할 수 있음.
* 따라서 `200 delay` 라는 로그는 200ms 뒤가 아닌 500ms 이후에 남게 됨.
* 그러므로 지정한 지연 시간 후에 정확히 Message가 처리된다고 가정하면 안됨.