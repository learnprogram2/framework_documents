[剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。例如，一个链表有6个节点，从头节点开始，它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个节点是值为4的节点。

```text
给定一个链表: 1->2->3->4->5, 和 k = 2.
返回链表 4->5.
```

思路: 快慢指针


```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        // 快慢指针, 栈

        ListNode fast = head, slow = null;
        // 1. fast 先往前走 k - 1 步 
        for (int i = 1; i < k; i ++) {
            // 预防链表不够长
            if (fast ==  null) {
                return null;
            }
            fast = fast.next;
        }
        // 2. 把slow顶上
        slow = head;
        // 3. 往后走
        while (fast.next != null) {
            fast = fast.next;
            slow = slow.next;
        }

        return slow;
    }
}
```