# N&M(1)

## 문제
자연수 N과 M이 주어졌을 때, 아래 조건을 만족하는 길이가 M인 수열을 모두 구하는 프로그램을 작성하시오.

1부터 N까지 자연수 중에서 중복 없이 M개를 고른 수열

## 입력
첫째 줄에 자연수 N과 M이 주어진다. (1 ≤ M ≤ N ≤ 8)

## 출력
한 줄에 하나씩 문제의 조건을 만족하는 수열을 출력한다. 중복되는 수열을 여러 번 출력하면 안되며, 각 수열은 공백으로 구분해서 출력해야 한다.

수열은 사전 순으로 증가하는 순서로 출력해야 한다.

##  풀이
```kotlin
import java.util.*


var list: IntArray = IntArray(9){0}
var check: BooleanArray = BooleanArray(9){false}
var m = 0
var n = 0

fun solve(count: Int) {
    if(count == m) {
        for(i in 0 until m)
            print("${list[i]} ")
        println()
        return
    }

    for(i in 1..n) {
        if(check[i])
            continue
        check[i] = true
        list[count] = i
        solve(count + 1)
        check[i] = false
    }
}
fun main() {
    var sc: Scanner = Scanner(System.`in`)

    n = sc.nextInt()
    m = sc.nextInt()

    solve(0)
}
``` 