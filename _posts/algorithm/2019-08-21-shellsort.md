---
layout: post
title: 希尔排序的实现
category: 算法探索
tags: sort
keywords: shell sort
---
   在介绍希尔排序之前需要先简单介绍一下简单插入排序，因为希尔排序可以算作是插入排序的
   变种，插入排序的原理是从序列的第二个元素开始，和前面的元素比较，找到比它小的元素，
   插在它后面（升序排列），相当于插在比它大的元素前面，然后循环至序列最后一个元素，
   每次要比较的元素前面都是有序的（所以简单插入排序是稳定的），而希尔排序是先将序列
   分成多个子序列对子序列进行插入排序，最后对整个序列进行一次插入排序(不稳定排序)。
   
   插入排序：
   ```
    待排序  5 2 3 1 6
    第一趟  2 5 3 1 6
    第二趟  2 3 5 1 6
    第三趟  1 2 3 5 6
    第四趟  1 2 3 5 6
   ```

   希尔排序：  
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2019-08-21.jpg"/>

JAVA实现代码：
```
import java.util.Arrays;

/**
 * Description: 希尔排序
 * Author: zhangliangfu
 * Create on: 2019-08-21 9:50
 */
public class ShellSort {

    /**
     * 插入排序
     * @param a
     */
    public static void insertSort(int a[]){
        for (int i = 1; i < a.length; i++) {
            int j;
            if (a[i]<a[i-1]){
                int temp = a[i]; //设置哨兵
                //集体往后移
                for (j = i-1; j>=0&&a[j]>temp; j--) {
                    a[j+1] = a[j];
                }
                a[j+1] = temp;
            }
        }
    }

    /**
     * 希尔排序
     * @param a
     */
    public static void shellSort(int a[]){
        int len = a.length;
        for (int distance = len/2; distance >= 1; distance/=2) {
            // 共有distance组
            for (int i = 1; i<=distance; i++){
                //分组进行插入排序
                for (int j = distance; j < len/distance; j+=distance) {
                    int k;
                    if (a[j]<a[j-distance]){
                        int temp = a[j]; //设置哨兵
                        for(k = j-distance; k>=0&&a[k]>temp; k-=distance){
                            a[k+distance] = a[k];
                        }
                        a[k+distance] = temp;
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        int a[] = {9, 6, 3, 7, 5, 1, 12, 14, 11};
//        insertSort(a);
        shellSort(a);
        System.out.println(Arrays.toString(a));

    }
}

```