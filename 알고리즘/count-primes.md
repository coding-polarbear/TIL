# Count Primes
## 문제
Count the number of prime numbers less than a non-negative number, n.

**Example:**
```
Input: 10
Output: 4
Explanation: There are 4 prime numbers less than 10, they are 2, 3, 5, 7.
```

## 풀이

```kotlin
class Solution {
    var resultList: ArrayList<Int> = ArrayList()
    fun countPrimes(n: Int): Int {
        val prime = BooleanArray(n)
        var count = 0
        
        for(i in 2 until n) {
            if(prime[i] == false) {
                count++;
                var j = 2;
                while(i * j < n) {
                    prime[i * j] = true
                    j++
                }
            }
        }
        return count
    }
}
```

에라토스테네스의 체를 이용해서 풀면 된다.

예전에 백준에서 풀었던 문제랑 비슷하게 할 수 있을 것 같다. 

i * j < n일동안 계속 돌면서 j의 배수를 없애준다.
