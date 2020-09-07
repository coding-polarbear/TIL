# Power of two
## 문제
> Given an integer, write a function to determine if it is a power of two.

**Example 1:**
```
Input: 1
Output: true 
Explanation: 20 = 1
```

**Example 2:**
```
Input: 16
Output: true
Explanation: 24 = 16
```

**Example 3:**
```
Input: 218
Output: false
```

## 풀이
```kotlin
class Solution {
    fun isPowerOfTwo(n: Int): Boolean {
        var  t = n
        var count = 0
        if(n <= 0)
            return false
        while(t != 0) {
            if(t % 2 == 1)
                count++
            if(count >= 2)
                return false
            
            t /= 2
        }
        return true
    }
}
```

어떤 수가 2의 제곱근인지 확인하는 문제이다.

여기서 유의해야할 점은,  `-16` 같은 수는 2의 제곱이 아니다.

그리고 한번이라도 2번 이상 2로 나눈 나머지가 0이 아니게되면, 그 수는 2의 제곱이 아니다.

