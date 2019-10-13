title: 剑指offer题解 | 链表中倒数第k个节点
author: Answer
tags:
  - 算法
categories:
  - 剑指offer
date: 2019-09-25 22:45:00
---
# 题目描述

输入一个链表，输出该链表中倒数第k个结点。


# 解题想法

- 设定两个标志位，low 和 high，high 先走 k-1 步，剩下的 low 和 high 一起走，high 等于 null 时就返回 low 节点

- 考虑链表长度为空，链表长度小于 k 的情况

	
# 代码实现


```

public ListNode FindKthToTail(ListNode head,int k) {
    if (head == null){
        return null;
    }
    ListNode low = head;
    ListNode high = head;
    for (int i = 0; i <= k-1; i++) {
        if (head == null && i < k){
        	// 链表长度小于 k
            return null;
        }
        high = head.next;
        head = head.next;
    }
    while (high != null){
        low = low.next;
        high = high.next;
    }
    return low;
}
```