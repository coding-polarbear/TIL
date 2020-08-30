# AsyncTask 클래스

## 백그라운드 스레드와 UI 스레드 구분

```java
public void onClick(View v) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            Bitmap bitmap = loadImageFromNetwork("http://example.com/image.png");
            mImageView.setimageBitmap(b);
        }
    }).start();
}
```

* 이 경우 백그라운드 스레드에서 UI를 변경하므로 `CalledFromWrongThreadExeception` 발생

### Handler 이용 (2가지 방식)

* Handler를 이용하는 2가지 방식 

**sendMessage() 메소드로 Message를 보내고 handleMessage() 메소드에서 UI 작업을 실행하는 것**
```java
private final static int BITMAP_MSG = 1;

private Handler mHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        if(msg.what == BITMAP_MSG) {
            mImageView.setImageBitmap((Bitmap) msg.obj);
        }
    }
};

public void onClick(Viw v) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            fiunal Bitmap bitmap = loadImageFromNetwork("https://example.com/image.png");
            Message message = Message.obtain(mHandler, BITMAP_MSG, bitmap);
            mHandler.sendMessage(message);
        }
    }).start();
}
```

**post() 메소드를 사용하여 Runnable에서 UI 작업을 실행하는 것**
```java
private Handler mHandler = new Handler();

public void onClcik(View v) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            final Bitmap bitmap = loadImageFromNetwork("https://example.com/image.png");
            mHandler.post(new Runnable() {
                @Override
                public void run() {
                    mImageView.setImageBitmap(bitmap);
                }
            });
        }
    }).start();
}
```

### View의 post() 메소드에 Runnable 전달
* 내부적으로 Handler를 사용함.
* Activity의 runOnUiThread() 메소드도 동일한 형태로 사용

```java
public void onClcik(View v) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            final Bitmap bitmap = loadImageFromNetwork("https://example.com/image.png");
            mImageView.post(new Runnable() {
                @Override
                public void run() {
                    mImageView.setImageBitmap(bitmap);
                }
            });
        }
    }).start();
}
```

### AsyncTask 이용
* 내부적으,로 Handler를 이용한 첫 번째 방식으로 되어 있음.

```java
public void onClick(View v) {
    new DownloadImageTask().execute("https://www.example.com/image.png");
}

private class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {
    @Override
    protected Bitmap doInBackground(String... urls) {
        return loadImageFromNetwork(urls[0]);
    }

    protected void onPostExecute(Bitmap result) {
         mImageView.setImageBitmap(result);
    }
}
```

### AsyncTask의 제네릭 파라미터
* `Params`, `Progress`, `Result`가 있음.
* 세개의 값이 모두 다 `Void` 인 것은 권장되지 않음.
* 최소한 `Params`에는 값이 전달해야 함.

## 액티비티 종료 시점과 불일치
* 액티비티에서 AsyncTask로 백그라운드 작업을 실행하는 중에 백키를 눌러서 액티비티를 종료하면
* 메모리에 액티비티가 남아있어서 모든 작업은 정상적으로 실행됨
* 다만 눈에 보이지 않음.


### 메모리 문제 발생 가능
* 단순히 백키를 눌렀을 때에는 `AsyncTask`가 오래 걸리는 것이 아니라면 일시적인 현상이므로 큰 문제는 되지 않음
* 다만 화면 회전 등으로 `AsyncTask`가 쌓여서 실행하는 경우 문제가 됨.
* `android:configChanges` 속성에 `orientation`이 들어 있는 것이 아니라면 화면이 회전할 때 액티비티는 종료되고 새로 시작함.
* 이때 새로 시작하는 액티비티는 다른 인스턴스인데 `AsyncTask`가 실행중이면 기존 액티비티도 메모리에서 제거되지 않음.
* 빈번하게 AsyncTask가 쌓이게되면 `OutOfMemoryException`이 발생할 수 있음.

### 순차 실행으로 인한 속도 저하
* 허니콤 이후에 `AsyncTask`는 순차 실행한다면 화면을 회전할 때 마다 작업이 쌓이므로 갈수록  실행이 느려질 수 있음.

### Fragment에서 AsyncTask 실행 문제
* AsyncTask 실행 도중 백키를 눌러 종료하면 `Fragment`에서 `getContext()`나 `getActivity()`가 null을 리턴함.
* `onPostExecute()` 에서 `Context`를 사용할 때 NPE가 발생하므로, 권장되는 방식은 `onPostExecute()` 메소드 시작 부분에서 `getContext()` 결과가 null이면 UI를 업데이트 하지 않고 리턴하는게 적절함.

## AsyncTask 취소
* `isCancelled()` 리턴값을 `doInBackground()` 곳곳에서 체크하고 AsyncTask 멤버 변수로 유지하고서 Activity의 `onDestroy()`에서 AsyncTask의 `cancel()` 메소드를 호출함.

### mayInterruptIfRunning 파라미터

`cancel(boolean mayInterruptIfRunning)`

* `doInBackground()`를 실행하는 스레드에 `interrupt()`를 실행할지 여부를 나타냄.
* true로 설정할 경우, 인터럽트를 체크해서 예외를 발생시킨다면 백그라운드 작업을 중지할 수 있음.
* false로 설정할 경우, 일부러 인터럽트를 주지 않고 내부적으로 백그라운드 작업을 계속 실행하는 게 유리할 때 쓸 수 있음.

## 예외 처리 메소드 없음
* AsyncTask에는 예외를 처리하기 위한 `onError()` 메소드가 없음

### AsyncTask의 기본 패턴 변현
* 백그라운드 스레드와 UI 스레드를 분리할때 백그라운드 스레드에서 예외 발생을 고려해야 함.
* 그러나 해당 내용이 AsyncTask에 없기 때문에 기본 패턴을 변형해서 사용하는 경우가 있음.

```java
public void onClick(View v) {
        new DownloadImageTask().execute("https://www.example.com/image.png");
   
}

private class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {
    @Override
    protected Bitmap doInBackground(String... urls) {
        try {
            return loadImageFromNetwork(urls[0]);
         } catch(Exception e) {
             return null;
         }
    }

    protected void onPostExecute(Bitmap result) {
        if(result == null)  {
            // 화면에 메시지를 보여줌.
            return;
        }
         mImageView.setImageBitmap(result);
    }
}
```

* 이방법도 완전한 방법은 아님
* `loadImageFromNetwork()`에 예외가 발생하지 않을 때에도 null을 리턴하는 경우가 있다면 에러 메시지가 의도에 맞지 않음.
#### 예외가 발생하면 Boolean.False 리턴
```java
public void onClick(View v) {
        new DownloadImageTask().execute("https://www.example.com/image.png");
   
}

private class DownloadImageTask extends AsyncTask<String, Void, Boolean> {
    private Bitmap bitmap;
    @Override
    protected Boolean doInBackground(String... urls) {
        try {
            bitmap = loadImageFromNetwork(urls[0]);
            return Boolean.TRUE;
         } catch(Exception e) {
             return Boolean.FALSE;
         }
    }

    protected void onPostExecute(Boolean result) {
        if(!result)  {
            // 화면에 메시지를 보여줌.
            return;
        }
         mImageView.setImageBitmap(bitmap);
    }
}
```

#### 대안으로 RxJava 사용
* AsyncTask에서는 예외 처리를 위해서 군더더기 코드가 생겨남.
* RxJava에서는 예외 처리 방법을 기본적으로 제공.

```java
public void onClick(View v) {
    Observable<Bitmap> observable = loadImageFromNetwork("https://example.com/image.png");
    observable.subscribeOn(Schedulers.io()) // loadImageFromNetwork() 가 실행되는 스레드 정함.
    .observeOn(AndroidSchedulers.mainThread()) // observer 메소드가 실행되는 스레드 정함.
    .subscribe(new Observer<Bitmap>() {
        @Override
        public void onNext(Bitmap bitmap) {
            mImageView.setImageBitmap(bitmap);
        }

        @Override
        public void onError(Throwable e) {
            // 화면에 에러 메시지를 보여준다.
        }

        @Override
        public void onComplete() {
            Log.d(TAG, "onComplete");
        }
    });
}
```

## 병렬 실행시 doInBackground가 실행 순서를 보장하지 않음.

* 버전이 올라가면서 AsyncTask가 병렬실행에서 순차실행응로 변경됨.

### 병렬 실행이 필요한 경우
* 화면에 필요한 여러 정보를 API로 한변에 가져오지 못할 때

### 병렬로 데이터를 가져올 때 데이터 간 의존성.
* 데이터간의 의존성이 있는 경우, 단순한 병렬 실행만으로 해결할 수 없음.
* 2개의 AsyncTask를 함께 시작하면 메인 스레드에서 동작하는 `onPostExecute()`는 단일 스레드 규칙에 의해 하나씩 실행되지만, 둘 중 어느 것이 `doInBackground()`를 먼저 끝내고 `onPostExecute()`를 실행하는지 알 수 없음.
* 결과가 나오는 것을 작업량으로 판단해서는 안되며, 스레드 간에 무엇이 먼저 실행된다고 가정해서도 안됨.
* 이런 문제를 제어하기 어렵다면 차라리 `순차 실행`을 하는 것이 나을 수도 있음.


### CountDownLatch로 실행 순서 조정.
* 2개의 AsyncTask간 병렬 실행도 하면서 실행 순서가 중요한 경우 `CountDownLatch`를 사용.

```java
private ArrayList<String> composedList = new ArrayList();
private CountDownLatch latch = new CountDownLatch(1);
private class AsyncTaskA extends AsyncTask<Void, Void, List<String>> {
    @Override
    protected List<String> doInBackground(Void... params) {
        return Arrays.asList("spring", "summer", "fall", "winter");
    }

    @Override
    protected void onPostExecute(List<String> result) {
        try {
            composedList.addAll(result);
            title.setText(TextUtils.join(", ", composedList));
        } catch(Exception e) {
            Toast.maketext(SomeActivity.this, "Error = " + e.getMessage(), Toast.LENGTH_LONG).show();
        } finally {
            latch.countDown();  //countDown()을 1회 실행할 때 마다 파라미터의 값이 1씩 줄어들고, 0이되면 대기 상태가 풀림.
        }
    }
}

private class AsyncTaskB extends AsyncTask<Void, Void, List<String>> {
    @Override
    protected List<String> doInBackground(Void... params) {
        try {
            return Arrays.asList("east", "south", "west", "north");
        } catch(Exception e) {
            Log.d(TAG, "exception : " + exception);
            return null;
        } finally {
            latch.await();      // 예외가 발생했을 때 AsyncTaskB가 먼저 실행되는 것 방지
        }
    }

    @Override
    protected void onPostExecute(List<String> result) {
        if(result != null) {
            composedList.addAll(result);
            title.setText(TextUtils.join(", ", composedList));
        }
    }
}


public void onClick(View view) {
    AsyncTaskCompat.executeParallel(new AsyncTaskA());
    AsyncTaskCompat.executeParallel(new AsyncTaskB());
}
```