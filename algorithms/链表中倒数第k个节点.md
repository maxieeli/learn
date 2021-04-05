# 链表中倒数第k个节点

## 题目描述

输入一个链表，输出该链表中倒数第k个节点

{1,2,3,4,5}, 1

{5}

## 解题思路

设链表的长度为N，设置两个指针P1和P2，先让P1移动K个节点。则还有N-K个节点可以移动。此时让P1和P2同时移动，可以知道当P1移动到链表结尾时，P2移动到N-K个节点处，该位置就是倒数第K个节点。

```java
public ListNode FindKthToTail(ListNode head, int k) {
  if(head == null)
    return null;
  ListNode P1 = head;
  while(P1 != null && k-- > 0) {
    P1 = P1.next;
  }
  if(k > 0) {
    return null;
  }
  ListNode P2 = head;
  while(P1 != null) {
    P1 = P1.next;
    P2 = P2.next;
  }
  return P2;
}
```