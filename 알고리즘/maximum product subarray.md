# Maximum Product Subarray
## 문제
> Given an integer array nums, find the contiguous subarray within an array (containing at least one number) which has the largest product.

**Example 1:**
```
Input: [2,3,-2,4]
Output: 6
Explanation: [2,3] has the largest product 6.
```
**Example 2:**
```
Input: [-2,0,-1]
Output: 0
Explanation: The result cannot be 2, because [-2,-1] is not a subarray.
```

## 풀이
```kotlin
class Solution {
    fun maxProduct(nums: IntArray): Int {
        var maxProduct = nums[0]
        var minProduct = nums[0]
        var answer = nums[0]
        if(nums.size == 1){
            answer = nums[0]
        } else {
            for(i in 1 until nums.size) {
                var now = nums[i];
                var a = maxProduct * now
                var b = minProduct * now
                maxProduct = Math.max(now, Math.max(a, b))
                minProduct = Math.min(now, Math.min(a, b))
                answer = Math.max(answer, maxProduct)
            }
        }
        return answer
    }
}
```

현재 값과 이전까지 곱중 최고 값을 비교하여 답을 구한다.
