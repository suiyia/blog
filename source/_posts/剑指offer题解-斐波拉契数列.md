title: 剑指offer题解 | 斐波拉契数列
author: Answer
tags: []
categories:
  - 剑指offer
date: 2019-09-23 20:39:00
---
# 题目描述

大家都知道斐波那契数列，现在要求输入一个整数 n，请你输出斐波那契数列的第 n 项（从 0 开始，第 0 项为 0）。
n<=39


# 解题想法

- 递归，f（n）= f（n-1）+ f（n-2），但是每一次的迭代计算结果没有保存，很容易栈溢出

- 循环方式，每次的结果进行累加

- 多项关系式递推

	
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
