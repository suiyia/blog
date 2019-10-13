title: 剑指offer题解 | 数值的整数次方
author: Answer
tags:
  - 算法
categories:
  - 剑指offer
date: 2019-09-24 22:22:00
---
# 题目描述

给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。

保证base和exponent不同时为0


# 解题想法

- 主要考察 指数函数 的特征以及程序的健壮性

- 底数不能为负数

- 底数为 0 时，0 的 n 次方始终为 0

- 指数为 0 时，且任意数的 0 次方都是 1

- 指数为负数时，结果是越来越小的

- 常规思路：循环累乘，时间复杂度 0（N）

- 递归思路：当n为偶数，a^n =（a^n/2）×（a^n/2），当 n为奇数，a^n = a^[(n-1)/2] × a^[(n-1)/2] * a 

	
# 代码实现


```
// 循环思路
public double Power(double base, int exponent) {
    if (base == 0 ){
        return 0;
    }
    if (exponent == 0){
        return 1;
    }
    boolean flag = false;
    if (exponent < 0){
        exponent = -1*exponent;
        flag = true;
    }
    double result = 1;
    for (int i = 0; i < exponent; i++) {
        result = result * base;
    }
    if (flag){
        return 1.0/result;
    }else {
        return result;
    }
}


// 递归思路
public double Power1(double base, int expone
    if (base == 0 ){
        return 0;
    }
    if (exponent == 0){
        return 1;
    }
    double result = 0.0;
    int n = Math.abs(exponent);
    result = Power1(base,n >> 1);
    result = result * result;
    if ((n & 1) == 1){
    	// 如果指数n为奇数，则要再乘一次底数base
        result = result * base;
    }
    if (exponent < 0){
    	// 如果指数为负数，则应该求result的倒数
        result = 1.0/result;
    }
    return result;
}



```

