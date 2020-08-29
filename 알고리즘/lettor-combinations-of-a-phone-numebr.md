# Letter Combinations of a Phone Number
## 문제
Given a string containing digits from 2-9 inclusive, return all possible letter combinations that the number could represent.

A mapping of digit to letters (just like on the telephone buttons) is given below. Note that 1 does not map to any letters.

**Example:**
```
Input: "23"
Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
```

**Note:**
* Although the above answer is in lexicographical order, your answer could be in any order you want.

## 풀이
```kotlin
class Solution {
    var numMap: HashMap<Int, String> = HashMap()
    var lengthMap: HashMap<Int, Int> = HashMap()
    init {
        numMap.put(2, "abc")
        numMap.put(3, "def")
        numMap.put(4, "ghi")
        numMap.put(5, "jkl")
        numMap.put(6, "mno")
        numMap.put(7, "pqrs")
        numMap.put(8, "tuv")
        numMap.put(9, "wxyz")
        
        lengthMap.put(2, 3)
        lengthMap.put(3, 3)
        lengthMap.put(4, 3)
        lengthMap.put(5, 3)
        lengthMap.put(6, 3)
        lengthMap.put(7, 4)
        lengthMap.put(8, 3)
        lengthMap.put(9, 4)
    }
    fun letterCombinations(digits: String): List<String> {
        var result: ArrayList<String> = ArrayList()
        if(digits.isEmpty()) return result
        
        var num = digits.substring(0, 1).toInt()
        if(digits.length == 1){
            return numMap.get(num)!!.toArrayList()
        }
        
        var s = letterCombinations(digits.substring(1, digits.length))
        
        for(i in 0 until lengthMap.get(num)!!) {
            for(j in 0 until s.size) {
                result.add(numMap.get(num)!!.substring(i, i+1).toString() + s[j])
            }
        }
        
        return result
    }
}

fun String.toArrayList(): ArrayList<String> {
    var list: ArrayList<String> = ArrayList()
    for(i in 0 until this.length) {
        list.add(this.substring(i, i+1))
    }
    return list
}
```

Backtracking 문제이다.
끝에서부터 하나씩 쪼개서 리스트를 만든 다음, 앞에 있는 내용과 더해서 리스트를 완성해나간다.