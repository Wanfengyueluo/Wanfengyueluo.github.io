---
title: 剑指Offer06-从尾到头打印链表
tags:
  - 剑指Offer
  - LeetCode
  - 链表
  - 栈
  - 递归
categories:
  - 剑指Offer
date: 2021-05-22 16:34:39
---

:page_facing_up:题目描述：

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

:thinking:思路：

思路一：

题意是对链表的倒序输出，自然会想到一种先入后出的数据结构——栈，将链表中的元素依次入栈，再出栈得出结果。

:coffee:代码：

```java
public int[] reversePrint(ListNode head) {
	// 使用栈，依次出栈
	ListNode cur = head;
	Stack<Integer> stack = new Stack<>();
	while (cur.next != null) {
		stack.push(cur.val);
		cur = cur.next;
	}
	/*
    * 如果在fori中有stack.size()，会因为pop()出栈操作而导致出循环的次数有变化
    * */
	int len = stack.size();
	int[] res = new int[len];
	for (int i = 0; i < len; i++) {
		res[i] = stack.pop();
	}
	return res;
}
```

思路二：

可以通过方法调用的顺序，使用递归

:coffee:代码：

```java
public int[] reversePrint(ListNode head) {
	// 2.尝试递归
	ListNode cur = head;
	ArrayList<Integer> list = new ArrayList<>();
	reverse(cur, list);
	int[] res = new int[list.size()];
	int index = 0;
	for (int num : list) {
		res[index++] = num;
	}
	return res;
}

private void reverse(ListNode head, ArrayList<Integer> list) {
	if (head != null) {
		reverse(head.next, list);
		list.add(head.val);
	}
}
```

