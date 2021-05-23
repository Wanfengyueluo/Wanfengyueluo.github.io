---
title: 剑指Offer24-反转链表
categories: [剑指Offer]
tags:
  - 剑指Offer
  - LeetCode
  - 链表
  - 双指针
---

:page_facing_up:题目描述：

定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

:thinking:思路：

> 思路参考 [吴师兄的图解剑指 Offer 结构化专栏](https://www.algomooc.com/347.html)

思路一：

使用三个指针，分别为 pre、cur、next，其中 pre 初始为 null，也是最终返回的链表头节点，cur 指向当前链表的头节点，next 为 cur 节点的下一个节点，遍历 cur 指针直至为 null，将 cur 的下一指针进行反转指向 pre ，然后将三个指针依次向后移动，最终返回 pre。

:coffee:代码：

```java
public ListNode reverseList(ListNode head) {
    if (head == null) {
        return null;
    }
    ListNode pre = null;
    ListNode cur = head;
    if (cur.next == null) {
        return cur;
    }
    ListNode next = cur.next;
    while (next != null) {
        cur.next = pre;
        pre = cur;
        cur = next;
        next = next.next;
    }
    cur.next = pre;
    pre = cur;
    return pre;
}

```

:coffee:代码写法二：

```java
public ListNode reverseList(ListNode head) {
    if (head == null) {
        return null;
    }
    ListNode pre = null;
    ListNode cur = head;
    if (cur.next == null) {
        return cur;
    }
    while (cur != null) {
        ListNode next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
    return pre;
}
```

:coffee:代码写法三:

```java
public ListNode reverseList(ListNode head) {
    ListNode q = null;
    while(head!=null){
        ListNode p = head;
        head = head.next;
        p.next = q;
        q=p;
    }
    return q;
}
```

> LeetCode 上有关于递归的解法，需要继续学习
