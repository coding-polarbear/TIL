# Handler Thread 클래스
* `Thread`를 상속하고, 내부에서 `Looper.prepare()`와 `Looper.loop()`을 실행하는 `Looper` 클래스
* `Looper`를 가진 스레드이면서 `Handler`에서 사용하기 위한 스레드
* `Handler`는 `HandlerThread`에서 만든 `Looper`에 연결

**HanlderThread 사용 방법**
```java
private HandlerThread handlerThread;

public Processor() {
    handlerThread = new HandlerThread("Message Thread");
    handlerThread.start();
}

public void process() {
    new Handler(handlerThread.getLooper()).post(new Runnable() {        // Handler의 세번째 생성자에 HandlerThread의  Looper를 전달. 이후에 이 Handler에서 보낸 Message가 HandlerThread에서 생성한 스레드에서 처리됨
        @Override
        public void run() {
            ...
        }
    });
}
```

## HandlerThread 프레임워크 소스
```java
public class HandlerThread extends Thread {
    Looper mLooper;

    @Override
    public void run() {
        Looper.prepare();
        synchronized(this) {
            mLooper = Looper.myLooper();        // getLooper()나 quit() 메소드에서 mLooper를 직접 사용하지 않음.
            notifyAll();                        // 대기하는 스레드를 깨움.
        }
        Looper.loop();
    }

    public Loopr getLooper() {
        if(!isAlive()) {        // Thread를 상속한 HandlerThread에서 start()를 호출했는지 체크. 
            return null;
        }
        synchronized(this) {
            while(isAlive() && mLoooper == null) {  // mLooper가 null인지 체크
                try {
                    wait();     //  대기
                } catch(InterruptedException e) {

                }
            }
        }
        return mLooper;
    }

    public boolean quit() {
        Looper looper = getLooper();
        if(looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }
}
```

## 순차 작업에 HandlerThread 적용
> UI와 관련 없지만 단일 스레드에서 순차적인 작업이 필요할 때 `HandlerThread` 사용

  * 체크 사태가 바뀔 때마다 스레들르 생성하겆나 스레드가 스레드를 가져다가 DB에 반영한다면?
    * Thread가 start()를 실행한 수서대로 실행되지 않기 때문에 DB에 반영할 때 최종 결과가 잘못 될 가능성이 있음.
    * 실행 순서를 맞춰야 하기 때문에 이때 `HandlerThread`를 사용
    * `HandlerThread`를 사용하지 않는다면, 
      * 백그라운드 스레드에서 무한루프를 만들고
      * BlockingQueue를 매개로 하여 반복문 내에서 가져오기(take)를 시 ㄹ행하여
      * 스레드 외부에서 put을 실행하면 됨.


**HandlerThread에서 메세지 처리**
```java
    private Handler favoriteHandler;
    private HandlerThread handlerThread;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        handlerThread = new HandlerThread("Favorite Processing Thread");
        handlerThread.start();      // HandlerThread 시작
        favoriteHandler = new Handler(handlerThread.getLooper()) {      // HandlerThread에서 만든 Looper를 Handler 생성자에 전달
            @Override
            public void handleMessage(Message msg) {        // Message를 받아서 DB에 반영
                MessageFavorite messageFavorite = (MessageFavorite) msg.obj;
                FavoriteDao.updateMessageFavorite(messageFavorite.id, messageFavorite.favorite);
            }
        };
    }

    private class MessageAdapter extends ArrayAdapter<Message> {
        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            ...
            holder.favorite.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    boolean checked = ((Checkbox) view).isChecked();
                    Message message = favoriteHandler.obtainMessage();
                    message.obj = new MessageFavorite(item.id, checked);
                    favoriteHandler.sendMessage(message);           // 체크 박스를 선택 / 해제 할 때마다 Handler에 Message를 보냄.
                }
            });
        }
    }

    @Override
    protected void onDestroy() {
        handlerThread.quit();   //  내부에서 Looper.quit()를 호출해서  Looper 를 종료함.
        super.onDestroy();
    }
```

