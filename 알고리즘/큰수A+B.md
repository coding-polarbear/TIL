# 큰수 A+B

##  문제
두 정수 A와 B를 입력받은 다음, A+B를 출력하는 프로그램을 작성하시오.

## 입력
첫째 줄에 A와 B가 주어진다.  (0 < A,B < 10^10000)

## 출력
첫째 줄에 A+B를 출력한다.

## 풀이
> 입력과 출력 전부 기본적인 정수 혹은 long의 단위를 초과하기 때문에, string으로 입력받아서 BigInteger로 변환해주었다.

```kotlin
import java.math.BigInteger
import java.util.*

fun main() {
    var sc = Scanner(System.`in`)
    var temp = sc.nextLine().split(" ")
    var input1 = BigInteger(temp[0])
    var input2 = BigInteger(temp[1])

    println(input1.add(input2).toString())
}
```