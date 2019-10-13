title: 剑指offer题解 | 合并两个排序的链表
author: Answer
date: 2019-09-25 23:37:08
tags:
---
# 题目描述

输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。


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