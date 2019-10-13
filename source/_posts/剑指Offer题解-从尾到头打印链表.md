title: 剑指offer题解 | 从尾到头打印链表
author: Answer
tags: 
  - 算法
categories:
  - 剑指offer
date: 2019-09-17 23:18:00
---
# 题目描述

输入一个链表，按链表从尾到头的顺序返回一个ArrayList。

时间限制：1秒 空间限制：32768K


# 解题想法

- 方法 1. 使用栈保存遍历链表的结果，然后 push 栈输出到 ArrayList

- 方法 2. 递归方式实现倒序输出



# 代码实现

- **递归方式实现**

```
	ArrayList<Integer> arrayList=new ArrayList<Integer>();

    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        if (listNode != null){
            printListFromTailToHead(listNode.next);
            arrayList.add(listNode.val);
        }
        return arrayList;
    }


    class ListNode {
        int val;
        ListNode next = null;

            ListNode(int val) {
            this.val = val;
        }
    }
```
