# Application
* `Application`도 `Activity`나 `Service`와 마찬가지로 `ContextWrapper`를 상속함.
* `Application`은 단독으로 시작하지는 않음.
* 앱 프로세스가 떠있지 않은 상태에서 다른 컴포넌트가 실행 요청을 받으면, 앱 프로세스가 생성되고 `Application`이 먼저 시작됨.
* 그 이후에 해당 컴포넌트가 실행됨.
* 즉, 액티비티, 서비스, 브로드캐스트 리시버, 콘텐트 프로바이더 가운데서 어느 것이든지 외부에서(앱 아이콘, 노티피케이션, 알람, 다른 앱) 실행을 요청할 경우, `Application`이 먼저 시작되어 있다면 바로 해당 컴포넌트를 시작함.
* `Application`의 `onCreate()`보다 `ContentProvider`의 `onCreate()`가 먼저 실행됨.

## 애플맄이션 인스턴스 가져오기
* Context만 전달된다면 `Context`의 `getApplicationContext()` 메소드로 언제든지 `Application` 인스턴스를 구할 수 있음.
* `Activity` 에서는특히 `getApplication()` 메소드를 사용해서 가져오기도 함.

## 앱 초기화
* `Application`은 다른 컴포넌트보다 먼저 실행되기 때문에 앱을 위한 초기화 작업을 `Application`의 `oncCreate()`에서 주로 실행함.
* `onCreate()` 메소드는 가능한 한 빨리 끝나야 함.
* 앱 아이콘을 클릭해서 액티비티를 새로 시작할 때 `Application`의 `onCreate()`에서 시작이 오래 걸린다면, 검은 화면이 오래보이거나 화면이 늦게 뜨는 현상이 발생할 수 있음.
* 이 때문에 `onCreate()`에서도 UI 블로킹을 최소화하기 위해 스레드에서 작업을 실행하기도 함.
* 백 키를 통해서 액티비티를 모두다 벗어나도 프로세스가 종료되지 않음.
* 이 상태에서 다시 앱 아이콘을 클릭하면 `Application`의 `onCreate()`가 실행되지 않음.

## Application은 프로세스에서 항상 유지되는 인스턴스
* `Application`은 앱 프로세스에서 메인 클래스인 `ActivityThread`에 `mInitialApplication`이라는 멤버변수로 있기 때무넹, 앱 프로세스가 살아있는동안 계속 유지되는 인스턴스
* 즉, 전역적인 앱 상태 (`global application state`)를 저장하기에 좋은 조건을 가지고 있음.

## Application에서 데이터 공유 문제
* 메모리가 부족하거나 다른 프로세스를 사용자에게 즉각적으로 반응해야할 때, 프로세스는 종료되었다가 재시작 하기도 함.
* 또 다른 케이스로는 마시멜로 이상부터 환경 설정에서 앱 퍼미션을 추가하거나 제거할 수 있는데 이때도 재시작함.
* 재시작하면서 `Application`의 `onCreate()`는 다시 실행되지만, 기존에 Application에 저장해놓은 데이터는 사라짐.
* 결론적으로 사라져버려도 문제가 없고 남아 있으면 유용한 캐시 같은 것이 `Application`의 데이터 공유에 적용되는 것이 좋음.