# Application 콜백
* `Application` 에서는 `onCreate()` 메소드만 오버라이드 하는 경우가 많지만, 다른 메소드도 유용한 경우가 있음.
* `Application`의 메소드 가운데서 `registerXxx/unregisterXxx` 메소드를 제외하면  `ComponentCallback2` 인터페이스의 3개 메소드가 있음.
* `Application`과 `Activity`뿐만 아니라 `Service`와 `ContentProvider`, `Fragment`도 `ComponentCallback2` 인터페이스를 구현하고 있음.

## ComponentCallback2 인터페이스
* `Application`에서 3개의 메소드는 ICS 이전에 비어 있는 메소드였음.
* ICS부터는 기본 동작이 `registerComponentCallbacks()` 통해 등록된 콜백의 메소드를 다시 호출하고 있음.
* 따라서 오버라이드할 때는 `super.onXxx()` 메소드도 호출해야 함.

### onConfigurationChanged(Configuration newConfig)
* 구성이 변경되면 `Application`, `Activity`, `Service`, `ContentProvider` 순으로 `onConfigurationChanged()` 메소드가 불림.
* `Application`의 `onConfigurationChanged()` 메소드가 가장 먼저 실행되므로, 구성 변경 시에 반드시 필요한 작업이 있다면 여기서 진행하면 됨.

## onLowMemory()
* 전체 시스템에 메모리가 부족해서 모든 백그라운드 프로세스가 강제 종료될 가능성이 있을 때 호출됨.
* 잡고 있는 캐시나 불필요한 리소스를 해제(release) 하는 작업을 하면 됨.
* ICS 이상에서는 `onTrimMemory()`에서 메모리 해제 작업을 하는 것을 권장함.
* `GC`는 `onLowMemory()` 리턴 이후에 실행됨.

## onTrimMemory(int level)
* ICS부터 사용 가능
* 파라미터에 전달하는 level에 따라 `onLowMemory()` 메소드보다 세분화해서 처리할 수 있음.
* `level`에는 `ComponentCallbacks2`의 상수 값이 전달도미.

## Application에 등록하는 콜백
* Application에 등록하는 콜백에는 3가지가 존재
* 모두 register / unregister 메소드가 있고, 여러개의 콜배을 등록할 수 있음.
  * Application.ActivityLifecycleCallbacks(ICS)
  * CoponentCallbacks(ICS)
  * Application.OnProviderAssistDataListener(젤리빈 API 레벨 18)

### ComponentCallbacks 인터페이스
* ICS이전에 `ComponentCallbacks`는 `Application`에 `onConfigurationChanged()`, `onLowMemory()~ 메소드로 있음.
* ICS 이상에서는 `onTrimMemory()` 메소드가 추가되고 콜백을 여러 개 등록할 수 있게 함
* `onTrimMemory()`는 `ComponentCallbacks`가 아닌 이를 상속한 `ComponentCallbacks2` 인터페이스에 있으므로 `onTrimMemory()`를 쓸 때 `registerComponentCallbacks()`에 `ComponentCallbacks2` 구현체를 넘기면 됨.

## ActivityLifecycleCallbacks 인터페이스
* `ActivityLifecycleCallbacks`에는 액티비티의 생명주기마다 대응하는 메소드가 있고, 액티비티 생명주기가 끝날 때마다 이들 메소드를 호출함.
* 모든 액티비티의 생명주기가 끝날 때마다 이들 메소드를 호출하고, 모든 액티비티의 생명주기 메소드에 동일한 작업을 적용할 때 사용할 수 있음.
