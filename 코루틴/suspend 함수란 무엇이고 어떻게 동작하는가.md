# Suspend 함수란 무엇이고, 어떻게 동작할까?

* `suspend` 키워드가 붙은 함수들은 `CPS(Continuation Passing Style)`로 사용될 수 있도록 `Continuation` 파라미터가 함수의 마지막 파라미터로 추가되며 반환값이 `Any?`로 변경됨.
* 이렇게 생성된 중단함수는 코루틴이나 다른 중담함수 안에서만 호출될 수 있다는 제약이 생김
* 코루틴이 제공하는 유용한 다른 중단함수들을 사용할 수 있게 된다는 장점도 갖게 됨.
* 중단 함수는 `suspend` 키워드 자체가 의미 하듯이 호출될 경우 코루틴의 실행을 멈추게 하는, 다시 말해 실행의 분절점이 될 수 있다는 것을 나타냄.
* `supsend` 키워드는 우리가 만든 어떤 함수에든 적용하여 해당 함수를 중단 함수로 만들 수 있음.
* `top-level function`, `extension function`, `member function`, `local function`, `operator function` 등
* 중단 함수가 코루틴에서 호출되면 그 시점에서의 실행정보들을 `Continuation` 객체로 만들어 캐시 해 두었다가 실행이 재개(`Resume`) 되면 저장된 실행 정보를 기반으로 실행을 다시 이어나가게 됨.

```kotlin
suspend fun sum(val1: Int, val2: Int): Int {
    delay(2000)
    return val1 + val2
}
```

> 이 함수는 중단 함수이므로 당연히 코루틴 혹은 다른 중단함숭 안에서 호출되어야 함.

```kotlin
fun main(args: Array<String>) = runBlocking {
    val job = GlobalScope.launch {
        val result = sum(1, 2)
        println("Result : $result")
    }
    job.join()
}
```

* 이제 이 코드를 빌드하면, 코틀린 컴파일러는 `sum` 함수를 아래 형식으로 변경함.

```
INVOKESTATIC com/smp/coroutinesample/basic/BasicSample4Kt.sum (IILkotlin/coroutines/Continuation;)Ljava/lang/Object;
```

* 위 형식에서 `IILkotlin/coroutines/Continuation` 부분은 이 sum 함수의 파라미터 형식을 나타내는데, 이 함수가 `I : Integer`, `I : Integer`, `L:  Object(with package name)` 이렇게 3개의 파라미터를 갖고 있음을 나타냄. (`Function signature`)
* 컴파일러가 함수의 가장 마지막 파라미터로 `Continuation object`를 임의로 추가했음을 알 수 있음.
* 이제 중단함수에 `Continuation` 이라는 추가 파라미터가 생겼으니 코루틴은 함수를 CPS(**Continuation** Passing Style) 형식으로 이용할 수 있게 되었고, 코루틴 프레임워크는 이를 통해 `suspend` / `resume` 전환 동작을 수행함.

```kotlin
fun main(args: Array<String>) = runBlocking {
    val job = GlobalScope.launch {
        repeat(2) { times ->
            longRunningTask(times, times + 1)
        }
    }
    job.join()
}

suspend fun longRunningTask(input1: Int, input2: Int): Int {
    delay(2000)
    println("Intermediate result has been calculated. : $input1")
    delay(2000)
    val result = input1 + input2
    println("All of the calculation process have done. : $result")
    return result
}
```

* `longRunningTask()` 중단함수는 두 번의 `delay()`함수 (`delay 역시 중단함수임`)을 호출하는 간단한 함수 (`여기서는 수행이 오래 걸리는 함수라고 가정함.`)
* 이 함수는 `main()` 함수에서 생성된 코루틴에서 두 번 수행됨.

![이미지1](https://miro.medium.com/max/700/1*Ekev19B35w5F-KHLoai5xw.png)

* 메인함수에서 시작된 `LongRunningTask1` (중단 함수)은 첫번째 `delay()` 함수를 만나면 일정 시간 그 수행을 멈춤 (`이를 중단점, suspension point 라고 부름`)
* 그렇게 되면 다른 멈춰있는 함수에게 실행 기회가 주어짐.
* `LongRunningTask2`가 수행되고 마찬가지로 `delay()`  함수를 만나 실행을 멈춤.
* 이런식으로 하나의 스레드 안에서 실행 시간을 분할해가며 수행되는 모습이 됨.
* 만약 메인 코루틴에서 각각의 중단 함수를 호출하지 않고 특정 중단함수가 또 다른 중단 함수를 호출할 수 있도록 변경한다 코루틴이 중첩될 때처럼 중단함수가 중첩될 때에도 `Continuation` 상태로 호출 정보가 저장되며 마지막 호출까지 완료되면 최초 호출 함수가 그 결과를 받을 수 있음.

![이미지2](https://miro.medium.com/max/700/1*eI3mDD8PgZXAl2FpOtw1ig.png)

* 위 그림을 보면 일반적인 함수 스택 구조와 비슷 해 보임
* 실제로 `Continuation`의 구현체들은 `ContinuationStackFrame` 이라는 인터페이스 또한 구현하는데 여기에는 호출자정보(`Caller stackframe`) 또한 가지고 있음.


## 결론
* 일반적인 함수의 호출은 운영체제게엇 그 호출 스택을 관리함.
* 코루틴은 코루틴 프레임워크가 `CPS` 방식으로 호출정보(`Continuation`)를 스택 형태로 유지하고 있다가 호출 스택의 가장 마지막 함수가 실행을 종료하면 결과 값이 직전 호출 함수들로 전파되며 직점 함수를 재개(`resume`)함.
* 만약 스택상의 어떤 함수가 예외를 발생시키면 예외 정보를 최초 호출함수까지 `Continuation`을 통해 전달함.