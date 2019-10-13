title: 剑指offer题解 | 二进制中 1 的个数
author: Answer
tags:
  - 算法
categories:
  - 剑指offer
date: 2019-09-23 22:57:00
---
# 题目描述

输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。


# 解题想法

- 位比较

- 思路1：使用**整数右移**与 1 进行位比较，计算总个数。但是如果是负数，高位右移将会补 1，所以可以使用 >>> 无符号右移或者将负数变为正数再进行右移。

- 使用 **1 左移**的方式进行位比较，一直比较到 Interger.MAX_VALUE = 0x7fffffff

- 思路2：如果一个整数不为 0，那么这个整数至少有一位是 1。

	如果我们把这个整数减 1，那么原来处在整数最右边的 1 就会变为 0，1 右边的 0 变为 1。其余所有位将不会受到影响。
    
    然后将两者相与，如果结果不为 0，说明原来整数最右边 1 的左边里面肯定包含 1，然后减 1 继续这么循环下去，不为 0 的次数就是 1 出现的次数。
    
    例：1100 & 1011 = 1000，数字 1100 最右边的 1 左边还有一个 1
    
	
# 代码实现


```
// 先变为正整数
public int NumberOf12(int n) {
    if (n == 0){
        return 0;
    }
    int count = 0;
    if (n < 0){
        n = n & 0x7fffffff;
        count ++;
    }
    while (n != 0){
        if ((n & 1) != 0){
            count++;
        }
        n = n >> 1;
    }
    return count;
}

// 整数无符号右移
public int NumberOf13(int n) {
    if (n == 0){
        return 0;
    }
    int count = 0;
    while (n != 0){
        if ((n & 1) != 0){
            count++;
        }
        n = n >>> 1;
    }
    return count;
}

// 1 左移
public int NumberOf1(int n) {
	if (n == 0){
    	return 0;
    }
    int count = 0;
    int flag = 1;
    while (flag != 0){
    	if (flag & n == 1){
        	count ++;
        }
        flag = flag << 1;
    }
    return flag;
}

// 思路 2
public int NumberOf11(int n) {
    if (n == 0){
        return 0;
    }
    int count = 0;
    while (n != 0){
        count ++;
        n = n & (n-1);
    }
    return count;
}


```

