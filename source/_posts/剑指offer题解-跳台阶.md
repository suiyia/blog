title: 剑指offer题解 | 跳台阶
author: Answer
tags:
  - 算法
categories:
  - 剑指offer
date: 2019-09-23 21:15:00
---
# 题目描述

一只青蛙一次可以跳上 1 级台阶，也可以跳上 2 级。求该青蛙跳上一个 n 级的台阶总共有多少种跳法（先后次序不同算不同的结果）。


# 解题想法

- 第 n 层台阶可能由：第 n-1 跳一阶过来，还可能由第 n-2 跳两阶过来，所以总情况 f（n）=  f（n-2）+ f（n-1）

- 其实就是斐波拉契数列

	
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
    return Fibonacci(n-1) + Fibonacci(n-2);
}
    
// 循环
public int Fibonacci23(int n){
	if (n == 0){
		return 0;
	}
	if (n == 1 || n == 2){
		return 1;
	}
	int a = 0;
	int b = 1;
	int c = 0;
	for (int i = 1;i<n;i++){
		c = a + b;
		a = b;
		b = c;
	}
	return c;
}

// 多项式递推
public int Fibonacci2(int n) {
    if(n==0) {
        return 0;
    } else if(n==1||n==2) {
        return 1;
    } else if(n==3) {
        return 2;
    } else {
        return 3*Fibonacci(n-3)+2*Fibonacci(n-4);
    }
}

```


# 注意点
