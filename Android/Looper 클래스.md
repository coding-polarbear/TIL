# Looper 클래스

## 스레드별로 Looper 생성

Looper는 `TLS(Thread Local Storage)`에 저장되고 꺼내어짐

* `ThreadLocal<Looper>`의 set 메소드로 새로운 Looper를 추가
* `ThreadLocal<Looper>`의 get 메소드로 쓰레드별로 다른 Looper를 가져옴
* `Looper.prepare()`에서 스레드별로 Looper를 생성
* 특히 메인 스레드의 메인 Looper는 `ActivityThread`의 `main()` 메소드에서 `Looper.prepareMainLooper()`를 호출하여 생성됨.
* `Looper.getMainLooper()`를 호출하면 어디서든 main looper를 가져올 수 있음.

## Looper별 Message Queue를 가짐.

* Looper는 각각의 `Message Queue`를 가짐.
* 특히 메인스레드에는 이를 통해 UI 스레드에서의 경합 상태를 해결
* 스레드별로 다른 큐를 사용 할 때는 Looper를 대신 사용하는 것이 더 단순할 수도 있음.

## Looper.loop() 메소드의 주요 코드
> Looper.loop() 메소드의 주요 코드를 살펴보자

```java
public static void loop() {
    fianl Looper me = myLooper));
    if(me == null) {
        throw new RuntimeException(
            "No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;
    for(;;) {
        Message msg = queue.next();     // MessageQueue에서 다음 Message를 꺼냄.
        if(msg == null) {               // Message가 null이면 리턴.
            return;
        }
        msg.target.dispatchMessage(msg);    // Message를 처리한다. target은 Handler 인스턴스 이고 결과적으로 Handler의 dispatchMessage() 메소드가 Message를 처리한다.
        msg.recycler();
    }
}

public void quit() {
    mQueue.quit(false);
}

public void quitSafely() {
    mQueue.quit(true);
}
```

MessageQueue에서 꺼낸 message가 null이 되는 시점은?

* Looper가 종료될 때.
* Looper가 종료하는 메소드는 `quit()`와 `quitSafely()` 두가지로, MessageQueue의 quit 메소드를 호출하고, 그 결과 `queue.next()`에서 null을 반환하고 for문이 종료됨.

## quit()와 quitSafely()의 차이

### quit()
* 아직 처리되지 않은 Message를 모두 제거

### quitSafely() 
* `sendMessageDelayed()` 등을 써서 실행 타임스탬프 뒤로 미룬 지연 Message를 처리. 
* `quitSafely()` 메소드를 실행하는 시점에서 현재 시간보다 타임스탬프가 뒤에 있는 Message를 제거하고 그 앞에 있는 Message는 계속해서 처리함.
* `quitSafely()`는 젤리빈(API 18) 이상에서 부터 사용할 수 있음. 