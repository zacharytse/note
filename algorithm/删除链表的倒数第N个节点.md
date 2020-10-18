[TOC]
#题目
给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。给定的n保证是一定有效的
#思路
1. 先用一个节点遍历前n个节点，则剩下len-n个节点未遍历。
2. 然后再用一个指针指向头结点，并设定一个该指针的前驱指针。
3. 3个指针再同时遍历，直到第一个节点指针为null，此时前驱指针指向了待删除节点的前驱，它的后继指向了该删除的节点。
这里要考虑删除头节点的情况，所以最后要判断一下前驱节点是否为空，为空则代表是要删除头节点
```java{.line-numbers}
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
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode node = head;
        ListNode cur = head;
        ListNode pre = null;
        int i = 0;
        while(i++ < n){
            node = node.next;
        }
        while(node != null){
            pre = cur;
            cur = cur.next;
            node = node.next;
        }
        if(pre == null){
            head = cur.next;
        } else {
            pre.next = cur.next;
        }
        cur = null;
        return head;
    }
}
```