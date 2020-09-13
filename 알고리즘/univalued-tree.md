# Univalued Tree
## 문제
A binary tree is univalued if every node in the tree has the same value.

Return true if and only if the given tree is univalued.

**Example 1:**

![이미지1](https://assets.leetcode.com/uploads/2018/12/28/unival_bst_1.png)

```
Input: [1,1,1,1,1,null,1]
Output: true
```

![이미지2](https://assets.leetcode.com/uploads/2018/12/28/unival_bst_2.png)

**Example 2:**

```
Input: [2,2,2,5,2]
Output: false
```

## 풀이
```kotlin
/**
 * Example:
 * var ti = TreeNode(5)
 * var v = ti.`val`
 * Definition for a binary tree node.
 * class TreeNode(var `val`: Int) {
 *     var left: TreeNode? = null
 *     var right: TreeNode? = null
 * }
 */
class Solution {
    fun isUnivalTree(root: TreeNode?): Boolean {
        if(root == null)
            return true
        
        if(root.left != null && root.left.`val` != root.`val`)
            return false
        
        if(root.right != null && root.right.`val` != root.`val`)
            return false
        
        return isUnivalTree(root.left) && isUnivalTree(root.right)
    }
}
```

left 혹은 right가 null이 아닐때, 값이 같지 않으면 false
그렇지 않으면 true를 리턴.
