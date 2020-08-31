# K-diff Pairs in an Array
## 문제
Given an array of integers and an integer k, you need to find the number of unique k-diff pairs in the array. Here a k-diff pair is defined as an integer pair (i, j), where i and j are both numbers in the array and their absolute difference is k.

**Example 1:**
```
Input: [3, 1, 4, 1, 5], k = 2
Output: 2
Explanation: There are two 2-diff pairs in the array, (1, 3) and (3, 5).
Although we have two 1s in the input, we should only return the number of unique pairs.
```

**Example 2:**
```
Input:[1, 2, 3, 4, 5], k = 1
Output: 4
Explanation: There are four 1-diff pairs in the array, (1, 2), (2, 3), (3, 4) and (4, 5).
```

**Example 3:**
```
Input: [1, 3, 1, 5, 4], k = 0
Output: 1
Explanation: There is one 0-diff pair in the array, (1, 1).
```

**Note:**
* The pairs (i, j) and (j, i) count as the same pair.
* The length of the array won't exceed 10,000.
* All the integers in the given input belong to the range: [-1e7, 1e7].

# 풀이
```kotlin
class Solution {
        fun findPairs(nums: IntArray, k: Int): Int {
        var count = 0
        nums.sort()
        var before = Int.MAX_VALUE
        println("test1 : " + (nums.filter{it != 0}.size > 0))
        if(k >= 0) {
            println("t1")
            for(n in nums) {
                if((k == 0 && n != before) && (nums.filter{it == n}.size > 1)) count += 1
                else if(nums.contains(n + k) && (n != before && k != 0))
                    count++
                before = n
            }
        }
        println("t2")
    
        return count
    }
}
```

* k == 0이고  nums 안에  해당 값이 2개 이상 있으면 count에 1을 더한다,.
* 그렇지 않은 경우, n+k가 존재할 경우 count에 1을 더한다.
  