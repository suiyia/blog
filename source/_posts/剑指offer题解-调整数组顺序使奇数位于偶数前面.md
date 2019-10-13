title: 剑指offer题解 | 调整数组顺序使奇数位于偶数前面
author: Answer
tags:
  - 算法
categories:
  - 剑指offer
date: 2019-09-24 23:40:00
---
# 题目描述

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分

并保证奇数和奇数，偶数和偶数之间的相对位置不变。


# 解题想法

- 插入排序

- 从左自右找到第一个偶数，然后在该位置右边找到第一个奇数，进行交换。

	
# 代码实现


```

public void reOrderArray(int [] array) {
    if (array.length == 0 || array.length == 1){
        return;
    }
    int i = 0;
    int j = 0;
    while (i < array.length){
    	// 找到第一个偶数
        while ((array[i] & 1) == 1){
            i++;
        }
        j = i + 1;
        while (j < array.length){
        	// 找到第一个奇数
            if ((array[j] & 1) == 0){
                j++;
            }else {
                break;
            }
        }
        // 插入排序
        if (j < array.length){
            int temp = array[j];
            for (int k = j-1; k >= i ; k--) {
                array[k+1] = array[k];
            }
            array[i++] = temp;
        }else {
            break;
        }
    }
}
```

