# EditText Focus Down

## 개요
간혹 `EditText`의 focus 상태에 따라 뷰가 변화 되어야 하는 경우가 있다. Google에서는 이를 위해 `OnFocusChangeListener()`라는 것을 만들어 두었다.

만약 focus가 키보드가 올라왔을때 잡히고, 키보드가 내려올 때 focus도 같이 변화한다면, 우리는 focus의 값을 가져와 비교적 쉽게 뷰를 제어할 수 있을 것이다. 문제는, 그렇지 않다는 점에 있다.

## 문제점
```kotlin
val onFocusListener : View.OnFocusChangeListener = View.OnFocusChangeListener { v, hasFocus ->
    if(hasFocus) {
        mVisibility.set(true)
    } else {
        mVisibility.set(false)
    }
}
```

focus의 상태에 따라 뷰의 `visibility`를 결정해주는 이런 코드가 있다고 치자. 이상적으로 동작하기 위해서, 키보드가 내려가거나 사용자가 뒤로가기 키를 눌러 키보드가 내려갔으면 focus는 해제되어야 한다.

그러나, 그럼에도 불구하고 커서는 계속 EditText에 남아있으며, View의 `visibility`는 변화하지 않았다.

## 어떻게 해결 할 것인가?
상위 루트뷰에 대해서 터치리스너를 걸어서 `EditText`가 아닌 곳을 터치하면 focus를 clear하는 방법도 있었지만, 해당 화면에서 발생하는 대부분의 터치 이벤트에 대해서 제어를 해야하기 때문에 비효율적이다.

찾아보니, 만약 EditText를 상속받아 오버라이딩하여 새로 구성한다면, 써볼만한 한가지 방법이 있었다. `onKeyPreIme()` 메소드를 오버라이딩하여, 인식된 key의 값이 `KeyEvent.KEYCODE_BACK`이면 focus를 clear 시키는 방법이었다.

마침 폰트 적용을 위해 EditText를 커스텀하여 사용하고 있던 상황에서, 이 방법은 잘 먹혔다.

```kotlin
override fun onKeyPreIme(keyCode: Int, event: KeyEvent?): Boolean {
    if(keyCode == KeyEvent.KEYCODE_BACK) {
        clearFocus()
    }
    return super.onKeyPreIme(keyCode, event)
}
```