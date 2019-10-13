title: 剑指offer题解 | 两个栈实现队列
author: Answer
tags: 
  - 算法
categories:
  - 剑指offer
date: 2019-09-19 21:13:00
---
# 题目描述

用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

时间限制：1秒 空间限制：32768K


# 解题想法

- 方法 1（常规思路，移动次数较多）

	- stack1 专门用于 push 数据，stack2 专门 pop 数据。
	
    - 调用 pop() 方法时候，stack1 数据全部 push 到 stack2，保存 stack2 最顶上元素。
    
    - pop 方法结束之前，将 stack2 的数据再推回到 stack1 中。


- 方法 2（推荐，移动次数少）
	
    - stack1 专门用于 push 数据。
    
    - pop 方法调用的时候，**若 stack2 为空**就先将 stack1 中的**全部数据**推到 stack2，然后 pop stack2 中的元素；否则直接 pop stack2 中的数据。



# 代码实现

- 方法二

```
	Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();

    public void push(int node) {
        stack1.push(node);
    }

    public int pop() {
        if (stack1.isEmpty() && stack2.isEmpty()){
            throw new RuntimeException("Empty exception!");
        }
        if (stack2.isEmpty()){
            while (!stack1.isEmpty()){
                stack2.push(stack1.pop());
            }
        }
        return stack2.pop();
    }
```


# 注意点

- pop 数据考虑边界情况，栈为空