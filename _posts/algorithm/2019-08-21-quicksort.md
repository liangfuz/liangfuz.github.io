---
layout: post
title: 快速排序的实现
category: 算法探索
tags: sort
keywords: quick sort
---
   快速排序和冒泡排序一样，都是属于交换排序，快速排序是基于分治思想的一种排序，其原理是先设定
   一个值作为分界值，而后从后往前，从前往后遍历，将大于该分界值的元素放在右边，小于的放在左边
   ，而后对左右两部分实施同样的操作，递归的完成整个排序。(由于是分治的，有可能破坏之前有序的元素
   所以属于不稳定排序)
   具体步骤如下：
   
   1.设定两个遍历指针，分别指向序列的首位和末尾，即初始时i=0,j=n-1;  
   2.以第一个元素作为分界值key，j从后往前遍历，找到第一个小于key的值a[j]即将a[i]与a[j]互换；  
   3.i从前往后遍历，找到第一个大于key的值a[i]即将a[i]与a[j]互换；  
   4.重复前两个步骤，直至i=j;  
   5.以a[i=j]元素作为分界值，分别对左边和右边部分重复上述步骤，递归执行直至不可分割，排序结束。  
   
   手绘流程图  
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2019-08-21-1.jpg"/>

Java实现代码：
```
import java.util.Arrays;

/**
 * Description: 快速排序
 * Author: zhangliangfu
 * Create on: 2019-08-21 15:45
 */
public class QuickSort {

    /**
     * 快排递归方法
     * @param a
     * @param start
     * @param end
     */
    public static void quickSort(int a[], int start, int end){
        int i=start,j=end;
        int key = a[i];
        int temp;
        while (i!=j){
            while (j>i){
                if (a[j]<key){
                    temp = a[i];
                    a[i] = a[j];
                    a[j] = temp;
                    break;
                }else {
                    j--;
                }
            }
            while (j>i){
                if (a[i]>key){
                    temp = a[i];
                    a[i] = a[j];
                    a[j] = temp;
                    break;
                }else {
                    i++;
                }
            }
        }
        //左侧部分还有数据
        if (i-1>start){
            quickSort(a, start, i-1);
        }
        //右侧部分还有数据
        if (i+1<end){
            quickSort(a, i+1, end);
        }
    }

    public static void main(String[] args) {
        int a[] = {5,3,7,6,4,1,0,2,9,10,8};
        quickSort(a,0,a.length-1);
        System.out.println(Arrays.toString(a));
    }
}

```