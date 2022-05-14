---
title: "快速排序"
share: false
categories:
  - 数据结构与算法
tags:
  - 算法
---

## 算法简述
- 不稳定排序
- 时间复杂度（平均）= O(nlog<sub>2</sub>n)
- 时间复杂度（最坏）= O(n<sup>2</sup>)
- 时间复杂度（最好）= O(nlog<sub>2</sub>n)
- 空间复杂度 = O(nlog<sub>2</sub>n)
## 原理讲解
应用分治法思想，使数组中的每个元素与基准值比较，数组中比基准值小的放在基准值的左边；大的放在右边；接下来在左部和右部分别递归地执行上面的过程直到排序结束。
1. 从数列中挑出一个元素，称为 “基准”（pivot)
2. 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作
3. 递归地把小于基准值元素的子数列和大于基准值元素的子数列排序
## 动图演示 
![title](https://i.bmp.ovh/imgs/2019/07/79f92499dcb7b5f3.gif)  

```c
#include<stdio.h>
#include<stdlib.h>
#define length(x) (sizeof(x)/sizeof((x)[0]))

void quicksort(int a[], int low, int high) {
    int start = low;
    int end = high;
    int key = a[start];

    while (start < end) {
        while(start < end && key <= a[end]) {
            end--;
        }
        if(key >= a[end]) {
            int tmp = a[end];
            a[end] = a[start];
            a[start] = tmp;
        }
        while(start < end && key >= a[start]) {
            start++;
        }
        if(key <= a[start]) {
            int tmp = a[start];
            a[start] = a[end];
            a[end] = tmp;
        }
        if(start > low) { quicksort(a, low, start-1); }
        if(end < high) { quicksort(a, end+1, high); }       
    }    
}

int main() {
    int a[10] = {12,20,5,16,15,1,30,45,23,9};
    int start = 0;
    int end = length(a)-1;
    quicksort(a, start, end);
    for(int i = 0;i < length(a); i++) {
        printf("%d ",a[i]);
    }
    return 0;
}
```
