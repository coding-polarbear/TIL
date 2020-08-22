#  median-of-two-sorted-arrays

## 문제
Given two sorted arrays nums1 and nums2 of size m and n respectively.

Return the median of the two sorted arrays.

Follow up: The overall run time complexity should be O(log (m+n)).

**Example 1**

```
Input: nums1 = [1,3], nums2 = [2]
Output: 2.00000
Explanation: merged array = [1,2,3] and median is 2.
```

**Example 2**

```
Input: nums1 = [1,2], nums2 = [3,4]
Output: 2.50000
Explanation: merged array = [1,2,3,4] and median is (2 + 3) / 2 = 2.5.
```

**Example 3**

```
Input: nums1 = [0,0], nums2 = [0,0]
Output: 0.00000
```

**Example 4**

```
Input: nums1 = [], nums2 = [1]
Output: 1.00000
```

**Example 5**

```
Input: nums1 = [2], nums2 = []
Output: 2.00000
```

## 풀이

```kotlin
class Solution {
    fun findMedianSortedArrays(nums1: IntArray, nums2: IntArray): Double {
        var nums3 = nums1 + nums2
        nums3.sort()
        var result: Double = 0.0
        var half = nums3.size / 2
        if(nums3.size % 2 == 0) {
            result = (nums3[half] + nums3[half - 1]).toDouble() / 2
            println(nums3.asList())
        } else {
            result = nums3[half].toDouble()
        }
        return result
    }
}
```

2개의 array를 합친 후, 정렬을 한다.


합쳐진 array의 길이가 홀수일 경우 중간 값은 `(array.size / 2)` 번째 값이다.


합쳐진 array의 길이가 짝수일 경우 중간 값은 `(arrays.size /2)` 번째 값과 `((arrays.size / 2) - 1)` 번째 값의 평균이다.


이를 더블 형태로 바꾸어준 후 리턴하면 된다.