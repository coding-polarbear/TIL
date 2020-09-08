# Power of Four
## 문제
> Given an integer (signed 32 bits), write a function to check whether it is a power of 4.

**Example 1:**
```
Input: 16
Output: true
```
**Example 2:**
```
Input: 5
Output: false
```

**Follow up: Could you solve it without loops/recursion?**

## 풀이
```kotlin
class Solution {
    fun isPowerOfFour(num: Int): Boolean {
        if(num <= 0)
            return false
        var t = num
        var count = 0
        while(t > 1) {
            if(count > 1)
                return false
            if(t%4 == 0) {
                t /= 4
            } else {
                count ++
            }
        }
        return true
    }
}
```

* 기본적으로, 언제 풀었던 Power Of 2와 비슷한 문제
* 4로 나누어 떨어지지 않은 count가 1 이상이면 바로 false 리턴 후 종료.

