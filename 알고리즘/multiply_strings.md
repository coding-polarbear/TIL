# Multiply Strings

Given two non-negative integers `num1` and `num2` represented as strings, return the product of `num1` and `num2`, also represented as a string.

**Note:** You must not use any built-in BigInteger library or convert the inputs to integer directly.

## Example 1:
```
Input: num1 = "2", num2 = "3"
Output: "6"
```

## Example 2:
```
Input: num1 = "123", num2 = "456"
Output: "56088"
```

## Constraints:
* `1 <= num1.length`, `num2.length <= 200`
* `num1` and `num2` consist of digits only.
* Both `num1` and `num2` do not contain any leading zero, except the number `0` itself.

## 풀이
```kotlin
class Solution {
    fun multiply(num1: String, num2: String): String {
        var result = IntArray(num1.length + num2.length)
        
        for(i in num1.length - 1 downTo 0) {
            for(j in num2.length - 1 downTo 0) {
                val n1 = Integer.valueOf(num1[i] - '0')
                val n2 = Integer.valueOf(num2[j] - '0')
                
                result[i + j + 1] += n1 * n2
            }
        }
        
        for(i in result.size -1 downTo 1) {
            result[i - 1] += result[i] / 10
            result[i] = result[i] % 10
        }
        
        var chk = false
        val stringBuilder = StringBuilder("")
        
        for(i in result.indices) {
            if(result[i] != 0 && !chk) chk = true
            if(chk) stringBuilder.append(result[i])
        }
        
        return if (stringBuilder.toString().length == 0) "0" else stringBuilder.toString()
    }
}
```