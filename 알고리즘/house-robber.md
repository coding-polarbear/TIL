# House Robber
## 문제
You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and it will automatically contact the police if two adjacent houses were broken into on the same night.

Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight without alerting the police.

 

**Example 1:**

```
Input: nums = [1,2,3,1]
Output: 4
Explanation: Rob house 1 (money = 1) and then rob house 3 (money = 3).
             Total amount you can rob = 1 + 3 = 4.
```

**Example 2:**

```
Input: nums = [2,7,9,3,1]
Output: 12
Explanation: Rob house 1 (money = 2), rob house 3 (money = 9) and rob house 5 (money = 1).
             Total amount you can rob = 2 + 9 + 1 = 12.
```

**Constraints:**

* 0 <= nums.length <= 100
* 0 <= nums[i] <= 400

## 풀이

```kotlin
class Solution {
    fun rob(nums: IntArray): Int {
        var last = 0
        var now = 0
        if(nums.size == 0) {
            return 0
        } else {
            var newLast = nums[0]
            var newNow = 0
            for(n in nums) {
                newLast = now + n
                newNow = Math.max(last, now)
                last = newLast
                now = newNow
            }
        }
        return Math.max(last, now)
    }
}
```

인접한 같은 집을 연달아서 털면 안된다.


처음에 짝수 / 홀수별로 나누어서 구할까라는 생각도 했었지만 첫번째 집을 택한 후 바로 4번째 집으로 넘어가는 경우도 있어서 그 방법은 쓸 수 없었다. 

따라서 0번째부터 더 큰 값을 만드는 조합을 구한다. 