---
title: 剑指Offer18-删除链表节点
tags:
  - 剑指Offer
  - LeetCode
  - 链表
categories:
  - 剑指Offer
date: 2021-05-22 16:35:25
---


:page_facing_up:题目描述：

给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。返回删除后的链表的头节点。

:thinking:思路：

思路：

用一个辅助指针对链表进行遍历，遇见待删除的元素时，将当前节点的`next`指针指向下下个节点。

:coffee:代码：

```java
public ListNode deleteNode(ListNode head, int val) {
    if(head.val==val) return head.next;
    ListNode p = head;
    while(p.next!=null){
        if(p.next.val==val){
            p.next = p.next.next;
            break;
        }
        p = p.next;
    }
    return head;
}
```

> 注意返回的是head而不是cur，因为cur指针相当于游标，相较于原head会有变化。
>
> 看别人的题解，会对节点进行判空操作。

