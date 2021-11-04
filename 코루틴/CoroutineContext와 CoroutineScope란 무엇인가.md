# CoroutineContext와 CoroutineScope란 무엇인가?

* 코루틴을 생성하기 위해서 `GlobalScope.launch{}` 블록을 이용하여 코드를 작성했고, `launch` 중단함수를 코루틴 빌더라고 이야기했음.
* 이름만 갖고 유추해보면, 가장 넓은 스코프 (`GlobalScope` = `ApplicationScope`)를 갖는 코루틴을 만들 수 있도록 도와주는 빌더 같음.
* 코루틴을 이해하기 위해서는 `CoroutineContext`와 `CoroutineScope`에 대해서 먼저 이해해야 함

## CoroutineContext의 구현 파일
```kotlin
public interface CoroutineContext {
  /**
   * Returns the element with the given [key] from this context or `null`.
   * Keys are compared _by reference_, that is to get an element from the context the reference to its actual key
   * object must be presented to this function.
   */
  public operator fun <E : Element> get(key: Key<E>): E?
  /**
   * Accumulates entries of this context starting with [initial] value and applying [operation]
   * from left to right to current accumulator value and each element of this context.
   */
  public fun <R> fold(initial: R, operation: (R, Element) -> R): R
  /**
   * Returns a context containing elements from this context and elements from  other [context].
   * The elements from this context with the same key as in the other one are dropped.
   */
  public operator fun plus(context: CoroutineContext): CoroutineContext = ...impl...
  /**
   * Returns a context containing elements from this context, but without an element with
   * the specified [key]. Keys are compared _by reference_, that is to remove an element from the context
   * the reference to its actual key object must be presented to this function.
   */
  public fun minusKey(key: Key<*>): CoroutineContext
}

/**
 * Key for the elements of [CoroutineContext]. [E] is a type of element with this key.
 * Keys in the context are compared _by reference_.
 */
public interface Key<E : Element>

/**
 * An element of the [CoroutineContext]. An element of the coroutine context is a singleton context by itself.
 */
public interface Element : CoroutineContext {
  /**
   * A key of this coroutine context element.
   */
  public val key: Key<*>
  
  ...overrides...
}
```
* `get()` : 연산자(`operator`) 함수로써 주어진 `key`에 해당하는 컨텍스트 요소를 반환함.
* `fold()` : 초기값(`initialValue`)을 시작으로 제공한 병합 함수를 이용하여 대상 컨텍스트 요소들을 병합한 후 결과를 반환함.
* `plus()` : 현재 컨텍스트와 파라미터로 주어진 다른 컨텍스트가 갖는 요소들을 모두 포함하는 컨텍스트를 반환함. 현재 컨텍스트 요소 중 파라미터로 주어진 요소에 이미 존재하는 요소는 버려짐.
* `minusKey()` : 현재 컨텍스트에서 주어진 키를 갖는 요소들을 제외한 새로운 컨텍스트를 반환함.
* Key 인터페이스
  * `Element` 타입을  제네릭   타입으로 가져야 함.
  * `Element`는 `CoroutineContext` 를 상속 
  * `key`를 멤버 속성으로 갖음.
  * ex)
    * `CoroutineId`
    * `CoroutineName`
    * `CoroutineDispatcher`
    * `CoroutineInterceptor`
    * `CoroutineExceptionHandler`
  * 요소들은 각각의 `key` 값을 기반으로 `CoroutineContext`에 등록됨.

* `CoroutineContext`는 인터페이스로써 3가지 종류의 구현체가 있음.
  * `EmptyCoroutineContext` : 특별히 컨텍스트가 명시되지 않을 경우 `Singleton` 객체가 사용됨.
  * `CombinedContext` : 두 개 이상의 컨텍스트가 명시되면 컨텍스트간 연결을 위해 컨테이너 역할을 함.
  * `Element` : 컨텍스트의 각 요소들도 `CoroutineContext`를 구현함.

## CoroutineScope
> 기본적으로 `CoroutineContext` 하나만 멤버 속성으로 정의하고 있는 인터페이스

```kotlin
public interface CoroutineScope {
    public val coroutineContext: CoroutineContext
}
```

* 우리가 사용하는 모든 코루틴 빌더들 (`launch`, `async`, `coroutineScope`, `withContext`)  등은 `CoroutineScope`의 확장 함수로 정의됨.
* 빌더들은 `CoroutineScope`의 함수들이고 이들이 코루틴을 생성할 때는 소속된 `CoroutineScope`에 정의된  `CoroutineContext`를 기반으로 필요한 코루틴들을 생성해나감.

### Andorid Activity에서의 예시
```kotlin
class MyActivity : AppCompatActivity(), CoroutineScope {
  lateinit var job: Job
  override val coroutineContext: CoroutineContext
  get() = Dispatchers.Main + job

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    job = Job()
  }
  
  override fun onDestroy() {
    super.onDestroy()
    job.cancel() // Cancel job on activity destroy. After destroy all children jobs will be cancelled automatically
  }

 /*
  * Note how coroutine builders are scoped: if activity is destroyed or any of the launched coroutines
  * in this method throws an exception, then all nested coroutines are cancelled.
  */
  fun loadDataFromUI() = launch { // <- extension on current activity, launched in the main thread
    val ioData = async(Dispatchers.IO) { // <- extension on launch scope, launched in IO dispatcher
      // blocking I/O operation
    }

    // do something else concurrently with I/O
    val data = ioData.await() // wait for result of I/O
    draw(data) // can draw in the main thread
  }
}
```

* `CoroutineScope` 인터페이스를 구현하고 있음.
* 그렇기 때문에 `CoroutineScope`가 정의하는 `CoroutineContext` 멤버 속성을 구현해야 함.
* 여기서는 `CoroutineScope`의 `CoroutineContext`를 `Dispatchers.Main + job` 으로 정의함으로써, `Activity`에서 생성되는 코루틴은 메인스레드(UI스레드)로 디스패치 되고, 액티비티에서 정의한 `Job` 객체를 `parent`로 하는 `job`들을 생성함으로써 액티비티의 `job`과 그 운명을 같이 하게 됨.

### GlobalScope.launch 인터페이스
* `CoroutineScope`는 지금까지 본 것과 같이 인터페이스일 뿐.
* 실제 구현체는 위에 예로 든 액티비티와 같이 어떠한 생명주기를 갖는 오브젝트에 적용하여 사용자 정의 스코프를 만들 수도 있지만, 편의를 위해서 코루튼 프레임워크에 미리 정의된 Scope들도 있음.
* 그중 하나가  `GlobalScope`

```kotlin
// -- in CoroutineScope.kt
object GlobalScope : CoroutineScope {
    override val coroutineContext: CoroutineContext
        get() = EmptyCoroutineContext
}

// -- in CoroutineContextImpl.kt
@SinceKotlin("1.3")
public object EmptyCoroutineContext : CoroutineContext, Serializable {
    private const val serialVersionUID: Long = 0
    private fun readResolve(): Any = EmptyCoroutineContext

    public override fun <E : Element> get(key: Key<E>): E? = null
    public override fun <R> fold(initial: R, operation: (R, Element) -> R): R = initial
    public override fun plus(context: CoroutineContext): CoroutineContext = context
    public override fun minusKey(key: Key<*>): CoroutineContext = this
    public override fun hashCode(): Int = 0
    public override fun toString(): String = "EmptyCoroutineContext"
}
```

* `Singleton object`로써 `EmptyCoroutineScope`를  컨텍스트로 갖고 있음.
* `EmptyCoroutineContext`는 구현해야할 모든 `CoroutineContext` 멤버 함수들에 대해서 기보 구현만 정의한 컨텍스트
* 기본 컨텍스트는 어떤 생명주기에 바인딩된 잡이 정의되어 있지 않기 때문에 애플리케이션 프로세스와 동일한 생명주기를 갖게 됨.
* `GlobalScope.launch{}` 로 실행한 코루틴은 애플리케이션 종료되지 않는 한 필요한 만큼 실행을 계속해 나감.