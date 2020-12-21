# AppWidgetProvider 클래스
* 앱 위젯은 브로드캐스트 리시버로도 만들 ㅅ수 있지만 대체로 `AppWidgetProvider`를 상속해서 만듬.
* `AppWidgetProvider`는 내부적으로 `onReceive()`에서 `Intent` 액션으로 구분한 후 `onUpdate()`, `onDeleted()`, `onEnabled()`, `onDisabled()` ;메소드로 `Intent extra` 값을 전달함.


```java
public class AppWidgetProvider extends BroadcastReceiver {
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if(AppWidgetManager.ACTION_APPWIDGET_UPDATE.equals(action)) {
            Bundle extras = intent.getExtras();
            if(extras != null) {
                int[] appWidgetIds = extras.getIntArray(
                    AppWidgetManager.EXTRA_APPWIDGET_IDS);
                if(appWidgetIds != null && appWidgetIds.length > 0) {
                    this.onUpdate(context, AppWidgetManager.getInstance(context), appWidgetIds);
                }
            }
        } else if(AppWidgetManager.ACTION_APPWIDGET_DELETED.equals(action)) {
            Bundle extras = intent.getExtras();
            if(extras != null && extras.containsKey(
                AppWidgetManager.EXTRA_APPWIDGET_ID)) {
                    final int appWidgetId = extras.getInt(AppWidgetManager.EXTRA_APPWIDGET_ID);
                    this.onDeleted(context, new int[]{appWidgetId});
                }
        } else if(AppWidgetManager.ACTION_APPWIDGET_OPTIONS_CHANGED.equals(action)) {
            Bundle extras = intent.getExtras();
            if(extras != null && extras.containsKey(AppWidgetManager.EXTRA_APPWIDGET_ID) &&
                extras.containsKey(AppWidgetManager.EXTRA_APPWIDGET_OPTIONS)) {
                    int appWidgetId = extras.getInt(AppWidgetManager.EXTRA_APPWIDGET_ID);
                    Bundle widgetExtras = extras.getBundle(AppWidgetManager.EXTRA_APPWIDGET_OPTIONS);
                    this.onAppWidgetOptionsChanged(context, AppWidgetManager.getInstance(context), appWidgetId, widgetExtras);
                }
        } else if(AppWidgetManager.ACTION_APPWIDGET_ENABLED.equals(action)) {
            this.onEnabled(context);
        } else if(AppWidgetManager.ACTION_APPWIDGET_DISABLED.equlas(action)) {
            this.onDisabled(context);
        }
    }

    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {

    }

    public void onAppWidgetOptionsChnaged(Context context, AppWidgetManager appWidgetManager, int appWidgetId, Bundle newOptions) {

    }

    public void onDeleted(Context context, int[] appWidgetIds) {

    }

    public void onEnabled(Context context) {

    }

    public void onDisabled(Context context) {

    }
}
```

* `appWidgetId`는 앱 위젯을 홈 스크린에 꺼낼 때 마다 새로 받는 인스턴스 id 값.
* 동일한 앱 위젯이 홈스크린에 여러 개 깔리더라도 인스턴스 id 값은 모두 다름.
* 즉, 앱 위젯의 종류별로 있는 값이 아니라 전역적인 값.
* 일반적으로 앱 위젯은 하나씩 제거되므로, `ACTION_APPWIDGET_DELETED` 액션에는 하나의 `appWidgetId`만 전달됨.


1) `ACTION_APPWIDGET_UPDATE` : 부팅 시, 최초 설치시, 업데이트 간격(`update interval`) 경과 시에 호출됨.
2) `ACTION_APPWIDGET_DELTED` : 인스턴스가 삭제될 때 호출됨.
3) `ACTION_APPWIDGET_OPTIONS_CHANGED` : 앱 위젯이 새로운 사이즈로 레이아웃 될 때 호출됨 (젤리빈 이상부터)
4) `ACTION_APPWIDGET_ENABLED` : 최초 설치시에 호출됨.
5) `ACTION_APPWIDGET_DISABLED` :  마지막 인스턴스가 삭제될 때에 호출됨.

* `ACTION_APPWIDGET_DELETED` 액션과 `ACTION_OPTIONS_CHANGED`, `ACTION_ENABLED`, `ACTION_DISABLED` 액션은 시스템에서만 보낼 수 있음 (protected intent);
* 앱에서는 `ACTION_APPWIDGET_UPDATE` 액션만 보낼 수 있음.
* `AppWidgetProvider`를 상속하면 `onReceive()` 메소드를 구현하지 않아도 되지만, Intent 액션을 별도로 사용한다면 `onReceive()` 메소드를 오버라이드해야 함.
* 앱 위젯에 버튼 클릭 같은 이벤트가 있어서 앱 위젯을 다시 그려야 하는 경우 등에 필요함.