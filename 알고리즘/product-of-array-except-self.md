# Product of Array Except Self
## 문제
Given an array nums of n integers where n > 1,  return an array output such that output[i] is equal to the product of all the elements of nums except nums[i].

**Example:**
```
Input:  [1,2,3,4]
Output: [24,12,8,6]
```
**Constraint**: It's guaranteed that the product of the elements of any prefix or suffix of the array (including the whole array) fits in a 32 bit integer.

Note: Please solve it without division and in O(n).

**Follow up:**
Could you solve it with constant space complexity? (The output array does not count as extra space for the purpose of space complexity analysis.)

## 풀이
```kotlin
class Solution {
    fun productExceptSelf(nums: IntArray): IntArray {
        var resultArray = IntArray(nums.size)
        
        for(i in 0 until nums.size) {
            var multi = 1
            for(j in 0 until nums.size) {
                if(i != j) {
                    multi *= nums[j]
                }
            }
            resultArray[i] = multi
        }
        return resultArray
    }
}
```