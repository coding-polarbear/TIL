# <activity-alias> 선언
* `<activity-alias>` : 액티비티의 별명

## 제거된 액티비티 대체
* `activity-alias`는 기존에 있던 액티비티가 소스에서 제거될 때 사용할 수 있음.
* 대체하는 화면이 존재한다면 activity-alais로 기존 액티비티 이름을 남겨두는 것을 고려 할 수 ㅣㅇㅆ음

## FLAG_ACTIVITY_CLEAR_TOP 플래그의 한계 해결
* 어떤 액티비티가 여러 개 있을때 해당하는 Activity를 시작하면서 `FLAG_ACTIVITY_CLEAR_TOP` 플래그를 사용하면 맨 위에 있는 액티비티만 클리어탑이 되어 나머지 여러개의 activity는 남아있게 됨
* 이때 activity-alias를 사용하고 별명으로 시작하게 되면 스택의 맨 아래 Activity 하나만 남기는 것이 가ㅡㄴㅇ함.

```xml
<activity-alias
    android:name=".FirstActivityA"
    android:targetActivity=".ActivityA"/>
```

```java
Intent intent = new Intent().setComponent(
    new Component(this, "com.someapp.FirstActivityA"));
intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP | Intent.FLAG_ACTIVITY_SINGLE_TOP);
startActivity(intent);
```
