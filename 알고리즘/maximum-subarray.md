# Maximum Subarray

# 문제
Given an integer array nums, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

**Example:**
```
Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
```

**Follow up:**
If you have figured out the O(n) solution, try coding another solution using the divide and conquer approach, which is more subtle.

## 풀이
dp 문제이다.

```kotlin
class Solution {
    fun maxSubArray(nums: IntArray): Int {
        var saveMaxList: ArrayList<Int> = ArrayList() 
        saveMaxList.add(nums[0])
        
        var max = nums[0]
        for(i in 1 until nums.size) {
            max = saveMaxList.get(i - 1) + nums[i]
            if(max < nums[i])
                max = nums[i]
            saveMaxList.add(max)
        }
        
        max = saveMaxList.get(0)
        for(i in 1 until nums.size) {
            if(max < saveMaxList.get(i)) 
                max = saveMaxList.get(i)
        }
        
        return max
    }
}
```

i-1 번째의 것과 현재 값을 더한 값이 max보다 클 경우 max 값을 더하는 식으로 문제를 풀어나갔다.
위의 방법에 따라 saveMaxList는 i번째에 올 수 있는 subArray의 max값이 저장되어있다.

따라서 저장된 값들 중에서 가장 큰 값이 우리가 찾고 잇는 가장 큰 subarray의 합이다.