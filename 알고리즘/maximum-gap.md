# maximum-gap
https://leetcode.com/problems/maximum-gap/

## 문제

Given an unsorted array, find the maximum difference between the successive elements in its sorted form.


Return 0 if the array contains less than 2 elements.


**Example 1**

```
Input: [3,6,9,1]
Output: 3
Explanation: The sorted form of the array is [1,3,6,9], either
             (3,6) or (6,9) has the maximum difference 3.
```

**Example 2**

```
Input: [10]
Output: 0
Explanation: The array contains less than 2 elements, therefore return 0.
```

**Note**

* You may assume all elements in the array are non-negative integers and fit in the 32-bit signed integer range.
* Try to solve it in linear time/space.


## 풀이

```kotlin
class Solution {
    fun maximumGap(nums: IntArray): Int {
        var maxGap = 0
        if(nums.size < 2) {
            return 0
        } else {
            nums.sort()
            for(i in 0 until nums.size - 1) {
                var temp = nums[i+1] - nums[i]
                if(temp > maxGap) {
                    maxGap = temp
                }
            }
        }
        return maxGap
    }
}
```

일단 최대의 gap을 구하기 위해서는 array가 정렬되어 있어야 편하다.


따라서 array를 정렬하고 시작한다. 


위의 예시에서 size가 2보다 작으면 0을 리턴했으므로, 거기에 대한 예외처리를 기본적으로 하고 시작한다.


i+1번째와 i번째 값을  비교하여 gap 차이가 현재의 maxGap보다 크면 maxGap 값을 바꾼다.