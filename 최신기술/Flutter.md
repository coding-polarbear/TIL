# Flutter, 어디까지 왔나?

# 모바일 앱 개발의 간략한 역사

### OEM SDK (Native Application)
> iOS나 Android 등 각 OS를 제조하는 회사들이 내놓은 SDK

Android는 JAVA (최근에는 코틀린), iOS는 objective-C(최근에는 Swift)를 기반으로 하고 있다.
* 앱은 위젯을 만들고 카메라와 같은 서비스에 접근하기 위해 플랫폼과 직접적으로 대화를 함.
* 이벤트에 대한 전달을 플랫폼에서 담당하기 때문에 비교적 간단한 구조
* 플랫폼마다 위젯과 언어가 다르기 때문에 각 플랫폼마다 독립적인 앱을 안들어야 함

### WebViews
> 하나의 코드로 여러개의 환경에서 동작하는 크로스 플랫폼 프레임워크를 향한 첫번째 시도

PhoneGap, Apache Cordova, Ionic등이 여기에 속함. Javascrpt와 WebView 기반.

* HTML을 만들고 이를 플랫폼 내부의 WebView를 통해 사용자에게 보여주는 방식
* Javascript가 NAtive code와 소통하는 것이 쉽지 않음
* 이를 해결하기 위한 Bridge 사용

### 리액티뷰 뷰
**Reactive Programming** 패턴을 이용한 **React**와 같은 리액티브 웹 프레임 워크는 웹뷰 생성을 매우 쉽게 만들어줬다는 점에서 큰 인기를 얻음

* React Native
  * Reactive 스타일의 뷰들이 가지고 있는 다양한 장점들을 모바일 앱에 적용하기 위해 만들어짐.
  * JavaScript 영역의 코드가 네이티브 영역에 있는 OEM 위젯에 접속하기 위해 결국 브릿지를 거치게 됨
  * OEM 위젯에 상당히 자주 접근 -> 퍼포먼스 이슈 야기

### Flutter
* React Native와 마찬가지로 리액티브 스타일 뷰 제공
* **Dart**라는 컴파일 프로그래밍 랭귀지를 이용해 브릿지로 인해 발생하는 성능문제 회피
* Flutter는 문맥 교환을 하는 JavaScript Bridge를 거치지 않고 플랫폼과 직접적으로 커뮤니케이션 함.


### Flutter는 무엇이 새롭고 흥미로운가?
* JavaScript 브릿지 없는 리액티브 뷰의 장점
* 빠르고, 부드럽고, 예측가능한 AOT에서 Naitive 코드로 컴파일되는 언어
* 위젯과 레이아웃에 모든 접근이 가능함
* Hot reload를 포함한 최고의 개발 환경
* 성능이 좋고, 호환성이 뛰어나며, 재미있음!

