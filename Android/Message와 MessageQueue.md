# Message와 MessageQueue

> MessageQueue는 Message를 담는 자료구조로, `LinkedBlockingQueue`에 더 가까운 구조.

* 배열 구조에 비해서 링크 구조는 일반적으로 개수 제한이 없고 삽입 속도가 빠름.
* 배열 구조를 랜덤 인덱스 접근이 가능하고 링크 구조는 순차 접근을 해야 함.
* `Message Queue`에는 `Message`가 실행 타임스탬프 순으로 삽입되고 링크로 연결되어, 실행 시간이 빠른 것부터 순차적으로 꺼내어짐.

## Message 클래스
```java
public final class Message implements Parcelable {
    public int what;
    
    public int arg1;

    public int arg2;

    public Object obj;

    public Messenger replyTo;

    ...

    /* package */long when;

    /* package */Bundle data;

    /* package */Handler target;

    /* package */Runnable callback;

    /* package */Message next;
}
```

* MessageQueue에 들얻가는 Message엣는 public 변수 int arg1, int arg2, Object obj, Messenger replyTo, int what 5개가 있음. Message를 만들 때 이변수에 값을 넣음.
* `Looper`, `Message`, `Message Queue`, `Handler`는 Message의 패키지 프라이빗 변수에 직접 접근함.
* target이나 callback 같은 것들이 Handler에서 postxxx(), sendXxx() 등을 호출할 때 Message에 담겨서 MessageQueue에 들어감.
* sendXxx(), postXxx() 메소드에서 실행시간(when)이 전달되고, 나중에 호출된 것이라도 타임스탬프가 앞서면 큐 중간에 삽입됨.
* 이것이 삽입이 쉬운 링크 구조를 사용한 이유

## obtain() 메소드를 통한 Message 생성
* Message를 생성할 때는 `오브젝트 풀(Object pool)`에서 가져오는 `Message.obtain()` 메소드나 Hanler의 `obtainMessage()` 메소드 사용을 권장.
* 내부적으로 Handler의 `obtainMessage()`는 `Message.obtain()` 메소드를 다시 호출함.
* 오브젝트 풀은 `Message`에 정적 변수로 있고 (여기서도 링크로 연결됨), Message를 최대 50개까지 저장함.
* Looper.loop() 메소드에서 Message를 처리하고 나서 recycerUnChecked() 메소드를 통해 Message를 다시 초기화하여 재사용함.
* 기본 생성자로 메시지를 생성해도 문제가 없어 보이지 불필요하게 pool에 메시지를 추가하면 금방 최대개수에 도달함.
* 따라서 따로 새엇ㅇ해서 풀에 돌려주면 자원을 낭비하게 됨.