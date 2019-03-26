# AAC MVVM과 List Adapter

## 개요
MVVM 패턴의 가장 큰 특징은 비지니스 로직이 ViewModel 안에 들어 있다는 것이다.

이때문에 기존 과정에서 한가지 문제가 발생하는데, `ListView` 혹은 `RecyclerView`에서 아이템에 대한 로직을 어디에 하냐는 것이다.

## ViewModel
![이미지](http://purplebeen.kr/images/1_8KprSpqqPtSuYObjOFPt2g.png)
MVVM(Model - View - ViewModel) 방식에 따라서, 비지니스 로직의  처리는 ViewModel에서 이루어져야 한다.

따라서, RecyclerAdapter에 있는 데이터에 대한 생성 / 로드 / 수정 / 삭제 로직 또한 ViewModel에서 이루어 지는 것이 맞다. `Adapter` 구조와 함께 생각해보면, `ViewModel`안에 리스트를 담이두고 데이터의 변화를 주는 메소드를 넣어 놓은 뒤, `View`에서 이를 `observe`하여 `Adapter`에 넣어주는 것이 합리적일 것이다. 

## AAC ListAdapter
이를 해결하기 위해서 AAC(Android Architecture Component)에서 새롭게 나온 것이 `ListAdapter`이다.

#### constructor
```kotlin
class MyListAdapter(diffUtil : DiffUtil.ItemCallback<Uri>) : ListAdapter<Uri, ViewHolder>(diffUtil)
```
생성자에서는 기존 `RecyclerAdapter`와 동일하게 제네릭에서 `ViewHolder`를 받고, 어떤 형태의 데이터를 사용할지 넣어준다.

어댑터의 객체를 만들 `View`에서는, `ViewModel`에서 `MutableLiveData<ArrayList<Uri>>`를 observe하여 `submitList()`로 `adapter`에 넣어준다.

```kotlin
mViewModel.mItemList.observe(this, Observer { 
    mMyAdapter.submitList(it)
})
```