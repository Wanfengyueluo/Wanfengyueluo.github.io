---
title: 剑指Offer22-链表中倒数第K个节点
tags:
  - 剑指Offer
  - LeetCode
  - 链表
  - 双指针
categories:
  - 剑指Offer
date: 2021-05-23 18:42:26
---


:page_facing_up:题目描述：

输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。

:thinking:思路：

思路一：

使用快慢两个指针，两个指针之间相差K个节点，当快指针指向链表尾端时，慢节点指向倒数第K个节点。

:coffee:代码：

```java
public ListNode getKthFromEnd(ListNode head, int k) {
    //  快指针
    ListNode fast = head;
    //  慢指针
    ListNode slow = head;
    while (fast != null) {
        fast = fast.next;
        if (k != 0) {
            k--;
        } else {
            slow = slow.next;
        }
    }
    return slow;
}
```

> 快慢指针的命名：
>
> 快 former
>
> 慢 latter
>
> 另一种写法可以先通过 K 步 for 循环找到快指针的位置，再进行 while 循环。

