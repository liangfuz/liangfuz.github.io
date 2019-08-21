---
layout: post
title: 归并排序的实现
category: 算法探索
tags: sort
keywords: merge sort
---
   归并排序是建立在规定操作上的一种排序算法，是采用分治法的一个典型的应用，其原理是先将序列
   分成多个子序列，然后使得子序列有序，之后再进行合并操作
   具体步骤如下：
   
   1.申请一个大小为两个子序列空间之和的新的空序列；  
   2.设定两个指针，分别指向两个子序列的起始位置；  
   3.比较两个元素，将较小的那个复制到新申请的空序列中，并将被复制的那个序列以及新的序列指针加1；  
   4.重复步骤3，两个序列中有一个序列指针超出序列尾；
   5.将指针未超出的那个序列所有剩下元素复制到新的序列尾部。
   
   PS：由于归并排序要求两个子序列是有序的，要做到这点就要把元素分割到最小粒度，那就是每组序列只有一个元素，
   因为当数组只有一个元素的时候它一定是有序的，之后再进行递归合并  

   手绘流程图  
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2019-08-21-2.jpg"/>

Java实现代码：
```
import java.util.Arrays;

/**
 * Description: 归并排序
 * Author: zhangliangfu
 * Create on: 2019-08-21 16:55
 */
public class MergeSort {
    public static void mergeSort(int[] a, int first, int last){
        if (first>=last){
            return;
        }
        int middle = (first+last)/2;
        mergeSort(a,first,middle);  //拆分左部分
        mergeSort(a,middle+1,last);  //拆分右部分
        merge(a, first, last, middle);  //比较并合并
    }

    public static void merge(int[] a, int f, int l, int m){
        int[] la = new int[m-f+1];
        int[] ra = new int[l-m];
        for (int i = 0; i < la.length; i++) {
            la[i] = a[f+i];
        }
        for (int i = 0; i < ra.length; i++) {
            ra[i] = a[m+1+i];
        }
        int i=0,j=0,k=f;  //i,j,k分别记录左右数组以及主数组的起始指针位置
        //将小的那一个复制到主数组
        while (i<la.length&&j<ra.length){
            if (la[i]<ra[j]){
                a[k] = la[i];
                i++;
                k++;
            }else {
                a[k] = ra[j];
                j++;
                k++;
            }
        }
        //有剩下的一并复制到主数组
        while (i<la.length){
            a[k] = la[i];
            i++;
            k++;
        }
        while (j<ra.length){
            a[k] = ra[j];
            j++;
            k++;
        }
        //复制end
    }

    public static void main(String[] args) {
        int a[] = {1,4,9,8,2,5,6,3};
        mergeSort(a,0,a.length-1);
        System.out.println(Arrays.toString(a));
    }
}

```