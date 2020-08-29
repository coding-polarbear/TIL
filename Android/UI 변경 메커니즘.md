# UI 변경 메커니즘

* UI를 변경하는 메소드는 주로 `set~~~` 으로 되어 있음.
* 커스텀 뷰를 만들면서 세터 메소드를 작성할 때, 단순히 `POJO(Plain Old Java Object)`와 같이 단순 대입만 하면 뷰가 변경되지 않음.
* 뷰를 다시 그리도록 `invalidate()` 메소드를 호출해야함.

```java
public void setTitle(String title) {
    this.title = title;
    invalidate()
}
```

* `invalidate()` 메소드를 호출하면
  * 메인 Looper의 `Message Queue`에 들어가서 다음 타이밍에 화면을 그림.
  * 이때 파라미터로 전달한 title을 `onDraw()` 에서 반영함.


## invalidate() 메소드의 호출 스택 확인.

View.invalidate() -> ViewGroup.invalidateChild(View child, final Rect dirty) -> parent.invalidateChildInParent(location, dirty) -> ViewRootImpl.invalidateChildInParent(int[] location, Rect dirty) -> scheduleTraversals()

* scheduleTraversals()는 무효화된 영역을 다시 그리기 위한 스케줄링


## invalidate()를 여러번 호출하는 경우

```java
public void onClick(View view) {
    for(int i = 0; i < 5; i++) {
        currentValue.setText("Current Value : " + i);
        SystemClock.sleep(1000);
    });
}
```

* 실제로 1초마다 텍스트가 변경되지 않는 이유는 5초동안 메인 스레드를 잡고 있기 때문에 화면 갱신이 5초동안 가능하지 않음.
* View에서 mPrivateFlags라는 플래그를 사용해서, 메소드 내에서 `invalidate()`를 여러 번 호출해도 한번만 `ViewRootImpl`에 전달됨
* 그렇다고 `invalidate()` 메소드가 계속 막혀서도 되지 않기 때문에 한번 그려진 후에 `invalidate()`가 호출이 되면 다시 그려짐.
* View와 ViewGroup, ViewRootImpl 에서 `mPrivateFlags`를 적절하게 변경하여 이문제를 해결 함.

## 다른 View 간에 invalidate() 메소드 호출

```java
public void onClick(View view) {
    title.setText("Go Go");
    image.setImageResource(R.drawable.icon);
}
```

* 이 경우 TextView와 ImageView를 변경하면 둘 다 `invalidate()` 메소드를 거쳐 `ViewRootImpl`에 도달하게 됨
* `ViewRootImpl` 에서 mTraversalScheduled 변수를 가지고 if무에서 체크해서 앞에서 한번 스케줄링 되었다면 다시 넣지 않음.