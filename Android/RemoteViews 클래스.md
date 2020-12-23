# RemoteViews 클래스
 * `RemoteViews`는 다른 프로세스에 있는 뷰 계층을 나타내는 클래스.
 * 알림(`Notification`)이나 앱 위젯에서는 앱에서 만든 레이아웃을 다른 프로세스에 보여줄 때 `RemoteViews`가 사용됨.
 * `RemoteViews`는 클래스명 때문에 일종의 `ViewGroup`으로 생각할 수도 있으나, `android.widget` 패키지 안에 있을 뿐 `View`나 `ViewGroup`을 상속한 것은 아님.
 * `RemoteViews`는 `Parcelable`과 `LayoutInflater.Filter` 인터페이스를 구현한 클래스에 불과함.

## RemoteViews에 쓸 수 있는 뷰 클래스
* `RemoteViews`의 레이아웃에 쓸 수 있는 뷰 클래스는 한정되어 있는데, `LayoutInflater.Filter` 인터페이스의 `onLoadClass()` 메소드가 뷰 클래스를 제안함.
* `LayoutInflater.Filter` 구현체인 `RemoteViews`에서 `booleanonLoadClass(Class clazz)` 메소드를 보면 클래스 선언에 `@RemoteView` 어노테이션이 있는 것만 `true`를 리턴함.
  * `FrameLayout`, `LinearLayout`, `RelativeLayout`, `GridLayout`
  * `AnalogClock`, `Button`, `Chronometer`, `ImageButton`, `ImageView`, `ProgressBar`, `TextView`, `ViewFlipper`
  * `ListView`, `GridView`, `StackView`, `AdapterViewFlipper`(허니컴부터 구현)
* 어노테이션은 상속된 클래스에는 적용되지 않기 때문에 위 목록에 있는 클래스의 하위클래스도 `RemoteViews`에는 쓸 수 없음.
* 커스텀 뷰도 쓸 수 없음. 클래스 선언에 `@RemoteViews`를 추가하면 될 것 같지만, 설치되는 프로세스의 런치에서 이 커스텀 뷰를 찾을 방법이 없음.
* 최상위 클래스인 `android.view.View`는 쓸 수 없음.
  * 레이아웃에 내용을 구분하기 위한 단순 라인 구분자를 만들 때 View에 배경색을 넣으면 됐지만, `RemoteViews` 에서는 `TextView`처럼 지원되는 뷰 클래스로 대체해야 함.
  * 이 경우 레이아웃에서  Lint 경고를 볼 수 잇는데 웹 위젯에서는 다른 방법이 없기 때문에 이런 경고는 무시해도 됨.
* `RemoteViews`는 뷰 계층상에 있는 레이아웃 리소스 아이디를 대상으로 작업을 다른 프로세스에 전달함.
* 내부적으로는 이런 작업 목록을 전달하고, `AppWidgetHost`에서 작업 목록을 한꺼번에 실행함.

## 뷰 클래스에서 사용 가능한 메소드
* `RemoteViews`에서 지원 가능한 뷰 클래스에서, 뷰를 변경하는 메소드가 제한적일 것 같지만 그렇지 않음.
* `RemoteViews` API 문서를 보면 메소드 가운데서 두 번째 파라미터에 `methodName` 문자열이 전달되는 것이 있는데, 리플렉션을 통해 세번째 파라미터 값을 두번째 파라미터인 `methodName` 이름의 메소드에 파라미터로 전달하게 됨.