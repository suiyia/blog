title: 剑指offer题解 | 反转链表
author: Answer
tags:
  - 算法
categories:
  - 剑指offer
date: 2019-09-25 22:44:00
---
# 题目描述

输入一个链表，反转链表后，输出新链表的表头。


# 解题想法

- 两个索引位保存链的指向信息，然后同步右移，原地反转

	
# 代码实现


```
	public ListNode ReverseList(ListNode head) {
       if (head == null){
            return null;
        }
        ListNode low = null;
        ListNode high = null;
        while (head != null){
            high = head.next;
            head.next = low;
            low = head;
            head = high;
        }
        return low;
    }
```