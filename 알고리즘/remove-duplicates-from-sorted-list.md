# Remove Duplicates from sorted list
## 문제

> Given a sorted linked list, delete all duplicates such that each element appear only once.

**Example 1:**
```
Input: 1->1->2
Output: 1->2
```

**Example 2:**
```
Input: 1->1->2->3->3
Output: 1->2->3
```


## 풀이
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode current = head;
        ListNode temp = head;
        while(current != null) {
            if(current.next == null) {
                temp.next = null;
                break;
            }
            
            if(current.val != current.next.val) {
                temp.next = current.next;
                temp = current.next;
            }
            current = current.next;
        }
        return head;
    }
}
```

오랜만에 자바로 풀었다/
현재의 val 값이 다음 val 값과 같지 않으면
next값을 다음걸로 변경해주는 식으로 풀었다.