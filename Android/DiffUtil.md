# DiffUtil
> DiffUtil is a utility class that can calculate the difference between two lists and output a list of update operations that converts the first list into the second one.

> DiffUtil은 두개의 리스트의 차이를 개산할 수 있고, 결과로 첫번째 리스트에서 두번째 리스트로 업데이트하는 동작을 수행할 수 있는 유틸리티 클래스이다.

어제 살펴봤던 것 처럼, AAC ListAdapter에는 `DiffUtil.ItemCallback<T>`가 생성자에 들어간다. 이것은 왜 들어가있을까?

일반적으로 리사이클러뷰 Adapter에서 리스트를 바꿔주는 경우, 뷰를 재구성 하는 과정이 부자연스럽고 시간이 눈에 보일정도로 꽤 소요되는 것을 볼 수 있다.

DiffUtil은 이 과정에서 2개의 리스트의 차이를 계산하여, 매끄럽게 리스트를 업데이트하고 애니메이션과 함께 뷰에 적용시킨다.


## DiffUtil.ItemCallback<T>의 메소드

#### areItemsTheSame(oldItem : T, newItem : T)
`같은 항목인가?`에 대하여 구분한뒤 같은 항목이면 `true`, 같은 항목이 아니면 `false`를 리턴한다.

좀더 구체적으로 예시를 들어보자.

이런 데이터 클래스가 있다고 가정해보자.
```kotlin
data class User(val id : Int, val message : String)
```

이 상황에서 `같은 항목인가?`의 구분은 `id`가 같은지 아닌지를 통해서 구분할 수 있다.

만약 여기서 `false`가 리턴된다면, 어댑터에서는 해당 아이템이 추가 또는 삭제되었다고 판단한다.

#### areContentsTheSame(oldItem : T, newItem : T) 

`같은 내용인가?`에 대하여 구분한 뒤 같은 내용이면 `true`, 같은 내용이 아니면 `false`를 리턴한다.

`areItemsTheSame()`에 있는 데이터클래스를 똑같이 예시로 생각해보면, `내용`이 같기 위해서는 `id`와 `message`가 모두  같아야 한다.

만약 여기서 `false`가 리턴된다면, 어댑터에서는 해당 아이템이 추가 / 삭제된 것이 아니라 `내용`이 변화되었다고 판단한다.