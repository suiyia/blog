title: 剑指offer题解 | 旋转数组的最小数字
author: Answer
tags: []
categories:
  - 剑指offer
date: 2019-09-19 23:27:00
---
# 题目描述

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。
NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。


# 解题想法

- 常规思路：从头到尾遍历一遍找到最小值，时间复杂度 O（N）

- 方法二
	 - 非递减数组旋转，旋转之后可以看做是两个非递减数组的拼接，当 a[index] > a[index+1] 时，最小值就是 a[index+1]。
     
     - 有序数组的查找，可以选择二分查找
     
     - **规律（数组内元素都不相同）**：若 A[mid] > a[low]，那么最小值一定在 mid 及 high 之间；若 A[mid] < a[low]，那么最小值一定在 low 及 mid 之间
     
- **注意点：如果 A[mid] = a[low]，那么就不能判断最小值在哪一个位置。例如 01111 旋转可以变为 11110 和 10111。只能从头到尾遍历**
	
# 代码实现

- 方法二实现

```
    public int minNumberInRotateArray(int [] array) {

        if (array.length == 0){
            return 0;
        }
        if (array.length == 1){
            return array[0];
        }

        int low = 0;
        int high = array.length - 1;

        while(low < high){
            int mid = (low + high) / 2;
            if (low == high -1){
                return array[high];
            }
            if (array[mid] > array[low]){
                // 最小值在右边
                low = mid;
            }else if (array[mid] < array[low]){
                // 最小值在左边
                high = mid;
            }else {
            	// 分辨不出最小值位置
  				return serach(array);
            }
        }

        return -1;
    }
    
    // 从头到尾查找一遍
    public int serach(int[] array){
        int minValue = Integer.MAX_VALUE;
        for (int i = 0; i < array.length; i++) {
            if (array[i] < minValue){
                minValue = array[i];
            }
        }
        return minValue;
    }
```


# 注意点

- 数组为空，或者只有一个长度的情况，考虑数组边界问题