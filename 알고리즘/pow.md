# Pow(x, n)

Implement pow(x, n), which calculates x raised to the power n (i.e. xn).

## 문제

**Example 1:**
```
Input: x = 2.00000, n = 10
Output: 1024.00000
```

**Example 2:**
```
Input: x = 2.10000, n = 3
Output: 9.26100
```

**Example 3:**
```
Input: x = 2.00000, n = -2
Output: 0.25000
Explanation: 2-2 = 1/22 = 1/4 = 0.25
```

**Constraints:**

* -100.0 < x < 100.0
* -231 <= n <= 231-1
* -104 <= xn <= 104


## 풀이

```kotlin
class Solution {
    fun myPow(x: Double, n: Int): Double {
        if(x == 0.0) return 0.0
        if(n == 0) return 1.0
        var half: Double = myPow(x, n/2)
        if(n % 2 == 0) return half * half
        else if(n > 0) return half * half * x
        else return half * half / x 
    }
}
```

x와 n에 대한 예외처리를 하고 시작한다.

n이 짝수인 경우, ex) n = 4인경우
x^4 = x^2 * x^2로 나타 낼 수 있다.

주기적으로  절반으로 나눈 값에  따라서, n이 짝수일때, n이 홀수 일 때, n이  0보다 작을 경우를 각각 계산하였다.