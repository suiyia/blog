title: 剑指offer题解 | 变态跳台阶
author: Answer
tags:
  - 算法
categories:
  - 剑指offer
date: 2019-09-23 21:52:00
---
# 题目描述

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法 


# 解题想法

- 参考前面的斐波拉契数列

- 第 n 层台阶可能由：第 n-1 跳一阶过来，还可能由第 n-2 跳两阶过来等，所以总情况 f（n）=  f（n-2）+ f（n-1）+ ... + f(1)

- 递推关系式实现 f（n）=  2 * f（n-1）

	
# 代码实现


```
// 递归版本
public int Fibonacci(int n) {

    if (n == 0){
        return 0;
    }
    if (n == 1){
        return 1;
    }
    return 2 * Fibonacci(n-1);
}
    
// 循环
public int Fibonacci23(int n){
	if (n == 0){
		return 0;
	}
	if (n == 1){
		return 1;
	}
	int b = 1;
	int c = 0;
	for (int i = 1;i<n;i++){
		c = 2 * b;
        b = c;
	}
	return c;
}

```


# 注意点
