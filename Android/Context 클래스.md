# Context 클래스
* Context가 없으면 액티비티를 시작할 수도, 브로드캐스트를 발생시킬 수도, 서비스를 시작할 수도 없음.
* Context는 여러 컴포넌트의 상위클래스이면서 Context를 통해서 여러 컴포넌트가 연결됨.
* 추상클래스로, 메소드 구현이 거의 없고 상수 정의와 추상 메소드로 이루어짐
* `ContextWrapper`가 `Context`를 직접 구현한 하위 클래스이며, `Activity`, `Service` 등은 `ContextWrapper`를 상속받음

## ContextWrapper 클래스
```java
Context mBase;
public ContextWrapper(Context base) { // 파라미터에 직접 전달되는 것은 Context의 여러 메소드를 직접 구현한 ContextImpl 인스턴스
    mBase = base;
}

protected void attachBaseContext(Context base) {    // 파라미터에 직접 전달되는 것은 Context의 여러 메소드를 직접 구현한 ContextImpl 인스턴스
    if(mBase != null) {
        throw new IllegalStateException("Base context already set");
    }
    mBase = base;
}

...

// ContextWrapper의 여러 메소드는 base의 메소드를 그대로 다시 호출함. 
// Activity, Service, Application 모두 내부적으로 ActivityThread에서 컴포넌트가 시작되는데, 이때 각 컴포넌트의 attach()메소드에서 attachBaseContext()를 호출함.
@Override
public Context getApplicationContext() {
    return mBase.getApplicationContext();
}

@Override
public void startActivity(Intent intent) {
    mBase.startActivity(intent);
}

@Override
public void sendBroacast(Intent intent) {
    mBase.sendBroadcast(intent);
}

@Override
public Resources getResources() {
    return mBase.getResources();
}
```

## ContextImpl은 컴포너트 별로 있음.

* `ContextWrapper`의 생성자에 전달되는 `ContextImpl`은 싱글톤으로 단 1개의 인스턴스만 존재하는게 아님.
* `Activity`, `Service`, `Application` 컴포넌트는 각각 생성한 `ContextImpl`을 하나씩 매핑하고 있고,  `getBaseContext()`는 각각 `ContextImpl` 인스턴스를 리턴함.
* `getApplicationContext()`는 `Application` 인스턴스를 리턴하는 것으로, 앱에서 1개밖에 없고 어디서나 동일한 인스턴스를 반환함.

## ContextImpl의 메소드
* 크게 헬퍼, 퍼미션, 시스템 서비스 3개 그룹으로 나눌 수 있음.
* 헬퍼 
  * 앱패키지 정보 제공
  * 내/ 외부 파일
  * SharedPreferences
  * 데이터베이스
* `Activity`, `Service`, `BroadcastReceiver`를 시작하는 메소드
* 퍼미션 체크 메소드
* `ActivityManagerService`를 포함한 시스템 서비스에 접근하기 위한 `getSystemService()` 메소드
  * `ContextImpl`의 정적 초기화 블록에서 클래스가 최초 로딩될 때 시스템 서비스를 매핑

## 사용 가능한 Context는 여러 개 있음.

액티비티를 예로 들면,

* `Activity` 인스턴스 자신
* `getBaseContext()`를 통해 가져오는 ContextImpl 인스턴스
* `getApplicationContext()`를 통해 가져오는 `Application` 인스턴스. 액티비티에서의 `getApplication()`과 같음.

**3개의 인스턴스가 다르기 때문에 캐스팅을 함부로 하면 안됨.**