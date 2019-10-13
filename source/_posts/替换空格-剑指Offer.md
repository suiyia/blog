title: 剑指offer题解 | 替换空格
author: Answer
tags: 
  - 算法
categories:
  - 剑指offer
date: 2019-09-17 21:31:00
---
# 题目描述

请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

时间限制：1秒 空间限制：32768K


# 解题想法


- 不使用工具类

- 思路：先统计空格出现次数，然后就知道了替换后的字符串长度，使用另外的字符串数组保存替换后的字符串，然后从右往左进行填充



# 代码实现


- 先统计空格出现的次数，出现一次空格，那么增加的长度就是加 2，N 个空格就是增加了 N*2

- 然后用另外的数组保存替换后的字符

```
public static String replaceSpace(StringBuffer str) {

		// 统计空格数量
        int spaceCount = 0;
        for (int i = 0; i < str.length(); i++) {
            if (str.charAt(i) == ' '){
                spaceCount++;
            }
        }
        
        int length = str.length();
        // 用另外的字符数组保存字符
        char[] a = new char[length + spaceCount * 2];
        
        // 字符数组索引，从最后面开始
        int afterIndex = length + spaceCount * 2 - 1;
        
        for (int j = length - 1; j >= 0; j--) {
            if (str.charAt(j) != ' '){
            	// 如果不是空格，就直接赋值，索引也跟着减
                a[afterIndex] = str.charAt(j);
                afterIndex --;
            }else {
                a[afterIndex] = '0';
                a[afterIndex - 1] = '2';
                a[afterIndex - 2] = '%';
                afterIndex = afterIndex - 3;
            }
        }
        return String.valueOf(a);
    }

```

# 注意点


- 从后往前填充相对简单点