# strStr

## 문제
Implement strStr().

Return the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.

**Example 1:**

```
Input: haystack = "hello", needle = "ll"
Output: 2
```

**Example 2:**

```
Input: haystack = "aaaaa", needle = "bba"
Output: -1
```


**Clarification:**

What should we return when needle is an empty string? This is a great question to ask during an interview.

For the purpose of this problem, we will return 0 when needle is an empty string. This is consistent to C's strstr() and Java's indexOf().


## 풀이
```kotlin
class Solution {
    fun strStr(haystack: String, needle: String): Int {
        var origin = haystack.toCharArray()
        var target = needle.toCharArray()
        var resultIndex:Int = -1
        
        var isCorrectAll:Boolean = false
        
        if(needle.isEmpty() && haystack.isEmpty()) resultIndex = 0
        else if(haystack.isEmpty()) resultIndex = -1
        else if(needle.isEmpty()) resultIndex = 0
        else if(origin.size < target.size) resultIndex = -1
        else {
            for(i in 0 until origin.size) {
                if(origin[i] == target[0]) {
                    for(j in 0 until target.size) {
                        if(((i + j) < origin.size) && (origin[i + j] == target[j]))
                            isCorrectAll = true
                        else {
                            isCorrectAll = false
                            break
                        }
                    }
                    if(isCorrectAll) {
                        resultIndex = i
                        break
                    }
                }
            }
        }
        return resultIndex
    }
}
```

c++의 `strStr()` 메소드를 구현하는 문제.

needle의 0번째와 현재 text의 i번째가 같은지 비교햐고,
i번째 기준으로 순서대로 needle과 동일한지 비교하여 startIndex를 구했다.