# 2019 Droidknight RxBinding Session 정리자료

# 지금까지 이런 간단한 Logic 처리는 없었다. 이것은 Rx인가 UI  이벤트인가? 네, RxBinding 입니다.

## RxBinding이란
> Android UI의 다양한 이벤트들을 Observable 형태로 바꾸어 주는 것

```kotlin
loginBtn.click().subscribe({
    requestAPI()
})
```

## RxJAVA를 사용하는 이유?
* 직관성
* 선언성
* 복합성
* 확장성
* 변화성

-> RxBinding을 사용하는 이유와 동일

### 예시
> 로그인 페이지를 만들어 달라는 요청 예시

**요구사항**
* 이메일 유효성에 맞지 않을 경우 에러메시지 표시
* 비밀전호는 8자리 이상 12자리 미만

## 요구사항은 멈추지 않는다.
> 이벤트 처리가 복잡하거나 여러가지 이벤트를 처리할 경우 로직이 복잡해지고 코드가 중첩된다. 

-> RxBinding을 이용하면 쉽게 해결 할 수 있다.

여러개의 이벤트들을 merge하여 `subscribe`하고, 이를 조립하듯이 사용할 수 있다.

## 어떻게 버튼 이벤트들이 Observable 형태로 바뀌었을까?
`subscribeActual() method`
* 메인스레드인지 확인하여 `onClickListener`를 상속받는 Listener에 뷰와 `Observer`를 넘겨준다.
* `Listener`에서는 observable의 `onNext()`를 호출한다.

## 실 적용 사례
복잡한 로직을 reactive로 해결하지 않고,
전역변수를 사용하면 로직이 꼬이거나 여러가지 문제가 발생할 수 있다.

예를 들어, `editText`의 텍스트 변화를 감지하려면 `TextWatcher   가 아니가 `textChanges()` 메소드를 이용하여 텍스트 변화를 감지할 수 있음

```kotlin
compositeDisposable.add(phoneCountryCodeChanges.subscribe {
    view.clearPhoneNumber()
})
```

## 주의해야 할 점
**RxJAVA.View**
* 생성된 `Observable`은 `view`에 대한 강력한 참조를 하고 있음
* 한번에 하나의 `Observable`만 view에서 사용할 수 있음 -> `.share()` 메소드를 이용하여 해결 가능

##  결론
> RxBinding을 이용하여 개발하면 코드를 함수형 프로그래밍 관점으로 보게 되고 어떤 상황이든지 좀 더 빠르게 대처할 수 있으며 무엇보다 콜백 지옥에서 빠르게 탈출할 수 있음.