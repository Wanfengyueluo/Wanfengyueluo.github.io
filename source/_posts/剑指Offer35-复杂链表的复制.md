---
title: 剑指Offer35-复杂链表的复制
tags:
  - 剑指Offer
  - LeetCode
  - 链表
  - 哈希表
categories:
  - 剑指Offer
date: 2021-05-25 17:56:53
---


:page_facing_up:题目描述：

请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

```java
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
```

:thinking:思路：

思路一：

> 思路参考 [吴师兄的图解剑指 Offer 结构化专栏](https://www.algomooc.com/354.html)

利用哈希表，第一次遍历原链表，通过哈希表将每个节点对应的新节点的位置进行存储，此时新复制的节点只包含val而不包含next和random指针。第二次遍历原链表，从原链表中取出每个新节点所对应的next和random指针。

:coffee:代码：

```java
public Node copyRandomList(Node head) {
    if (head == null) {
        return null;
    }
    Map<Node, Node> map = new HashMap<>();
    Node cur = head;
    while (cur != null) {
        map.put(cur, new Node(cur.val));
        cur = cur.next;
    }
    cur = head;
    while (cur != null) {
        map.get(cur).next = map.get(cur.next); //这里map.get(cur.next)就对应新复制的节点。
        map.get(cur).random = map.get(cur.random);
        cur = cur.next;
    }
    return map.get(head);
}
```

思路二：

> 思路参考 LeetCode 题解中K神的题解

利用辅助链表，即在遍历原链表时，将每个节点进行复制，然后在进行拆分，如`1->2->3->null`==>`1->1->2->2->3->3->null`

:coffee:代码：

```java
public Node copyRandomList(Node head) {
    if (head == null) {
        return null;
    }
    Node cur = head;
    while (cur != null) {
        Node node = new Node(cur.val);
        node.next = cur.next;
        cur.next = node;
        cur = node.next;
    }
    cur = head;
    while (cur != null) {
        if (cur.random != null) {
            cur.next.random = cur.random.next;
        }
        cur = cur.next.next;
    }
    cur = head.next;
    Node pre = head;
    Node res = head.next;
    while (cur.next != null) {
        pre.next = pre.next.next;
        cur.next = cur.next.next;
        pre = pre.next;
        cur = cur.next;
    }
    pre.next = null;
    return res;
}
```

> 先复制节点，然后找到节点对应的random指针，最后将整个链表拆分

