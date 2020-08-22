# reverse-words-in-a-string
https://leetcode.com/problems/reverse-words-in-a-string/

## 문제
Given an input string, reverse the string word by word.

**Example 1**

```
Input: "the sky is blue"
Output: "blue is sky the"
```

**Example 2**

```
Input: "  hello world!  "
Output: "world! hello"
Explanation: Your reversed string should not contain leading or trailing spaces.
```

**Example 3**

```
Input: "a good   example"
Output: "example good a"
Explanation: You need to reduce multiple spaces between two words to a single space in the reversed string.
```

**Notes**

* A word is defined as a sequence of non-space characters.
* Input string may contain leading or trailing spaces. However, your reversed string should not contain leading or trailing spaces.
* You need to reduce multiple spaces between two words to a single space in the reversed string.

## 풀이

```kotlin
class Solution {
    fun reverseWords(s: String): String {
        var strArray = s.trim().replace("\\s{2,}".toRegex(), " ").split(" ")
        var result = ""
        for(i in strArray.size-1 downTo 0) {
            result += strArray[i]
            if(i != 0)
                result += " "
        }
        return result
    }
}
```

string에서 trim()을 이용해 양쪽 끝 공백을 제거하고, 


정규표현식을 이용해 2칸 이상의 공백을 1개짜리 공백으로 치환한 뒤


역순으로 string을 넣어주었다.