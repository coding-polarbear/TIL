# Android Material Ripple Effect

## 개요
안듣로이드를 위한 앱을 제작하다보면, 디자인상 혹은 다른 이유때문에 기본적으로 제공하는 버튼 컴포넌트가 아니라 커스텀한 버튼을 적용하는 경우가 종종 있다. 그러나 커스텀 버튼 리소스를 만들때 `ripple`을 따로 설정해주지 않으면 버튼은 눌렸을 때와 눌리지 않았을 때가 동일하게 된다. 이는 background의 컬러만 바꾸었을때에도 동일하다.

> 그렇다면, 간단하게 사용자가 버튼이 눌렸다는 것을 알 수 있게 하는 방법은 없을까?

## Android Material Button Default Background
 안드로이드에는 `API level 21` 부터 Ripple Effect를 제공하는 몇가지 기본 Background가 있다. 
 
```xml
?android:attr/selectableItemBackground
```

```xml
?android:attr/selectableItemBackgroundBorderless
```
이 2개가 주인공이다.

`selectableItemBackground`는 버튼의 크기만큼 border로 잡아 버튼을 눌렀을 경우 `Ripple Effect`를 생성한다.

`selectableItemBackgroundBorderless`는 버튼의 테두리 없이 둥그렇게 처리하여 `Ripple Effect`를 발생 시킨다.

## 문제점
이렇게 하였을때 가장 큰 문제점은 대부분의 커스텀 버튼들은 백그라운드를 커스텀하여 발생한다는 것이다. 백그라운드에 위의 내용들을 넣게 되면 버튼을 커스텀할 수 없게 된다.

## 해결책
**foreground** 속성

`Android API level 21`부터 **foreground**라는 속성이 생겼다.
이는, 현재 뷰에 오버레이를 발생시켜 뷰의 앞부분에 덮여 씌운다. 

따라서, 위에서 언급했던 `?android:attr/selectableItemBackground`나 `?android:attr/selectableItemBackgroundBorderless`를 **foreground**에 적용한다면 보다 쉽게 Ripple Effect를 적용할 수 있을 것 이다.