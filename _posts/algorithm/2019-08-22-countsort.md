---
layout: post
title: 计数排序的实现
category: 算法探索
tags: sort
keywords: count sort
---
   计数排序是一种非比较排序算法，元素从未排序状态变成已排序状态的过程是由额外的辅助空间和元素本身
   的值决定的  
   排序步骤：
   1.根据待排序集合中最大值与最小值的差值申请额外的空间；  
   2.遍历待排序的集合，将元素放到元素值对应的下标的新申请的空间里，并累计次数；  
   3.将新申请的空间里非零的元素取出，元素的下标就是值，元素的值则代表出现了多少次，
   按下标顺序对应放回旧的数组中  

   PS:利用类似哈希地址的方式加上计数的逻辑完成的排序，比所有的比较排序都要快，但是需要
   空间，是以空间换时间的做法，当元素值分布不集中的时候会耗费大量内存。
   
   手绘流程图 
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2019-08-22.jpg"/>
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2019-08-22-1.jpg"/>

JAVA实现代码：
```
import java.util.Arrays;

/**
 * Description: 计数排序
 * Author: zhangliangfu
 * Create on: 2019-08-22 10:48
 */
public class CountSort {
    /**
     * 计数排序
     * @param a
     */
    public static void countSort(int a[]){
        int max = a[0];
        int min = a[0];
        //找出最大最小值
        for (int i = 0; i < a.length; i++) {
            if (a[i]<min){
                min = a[i];
            }
            if (a[i]>max){
                max = a[i];
            }
        }
        //申请空间
        int b[] = new int[max-min+1];
        //放到对应下标并计数
        for (int i = 0; i < a.length; i++) {
            int num = a[i]-min;
            b[num] += 1;
        }
        int k = 0;
        //得到排序后的数组
        for (int i = 0; i < b.length; i++) {
            if (b[i]>0){
                for (int j = 0; j < b[i]; j++) {
                    a[k] = i + min;
                    k++;
                }
            }
        }
    }

    public static void main(String[] args) {
        int a[] = {3,-1,2,8,5,6,3,2,4,9,7};
        countSort(a);
        System.out.println(Arrays.toString(a));
    }
}
```