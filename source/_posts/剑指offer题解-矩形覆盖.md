title: 剑指offer题解 | 矩形覆盖
author: Answer
tags:
  - 算法
categories:
  - 剑指offer
date: 2019-09-23 22:49:00
---
# 题目描述

我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？


# 解题想法

- 实质是斐波拉契数列

	
# 代码实现


```
// 递归版本
    public int RectCover(int target) {
        if (n == 1)
            return 1;
        if (n == 2)
            return 2;
        return RectCover(n - 1) + RectCover(n - 2) * 2;
    }
   

```


# 注意点
