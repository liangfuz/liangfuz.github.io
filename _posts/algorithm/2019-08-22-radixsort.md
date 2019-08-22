---
layout: post
title: 基数排序的实现
category: 算法探索
tags: sort
keywords: count sort
---
   基数排序是一种分配式排序，又称“桶子法”，是通过元素值的部分信息，将元素分配至某些“桶”中
   以达到排序的作用，例如按数字每个位数上的值分配至对应的桶中，循环分配直至所有位数都排完，
   最后获得有序的序列，基数排序是稳定排序。  
   基数排序有两种方式实现：  
   低位优先，LSD，Least Significant Digit first  
   高位优先，MSD，Most Significant Digit first  
   
   以LSD为例解释基数排序的步骤：
   ```
    待排序序列：9,5,14,4,21,46,233,61
    个位       十位       百位               
    0          0 4 5 9   0 4 5 9 14 21 46 61   
    1 21 61    1 14      1         
    2          2 21      2 233     
    3 233      3 233     3         
    4 14 4     4 46      4         
    5 5        5         5         
    6 46       6 61      6         
    7          7         7         
    8          8         8         
    9 9        9         9     
    最后得出排序后的序列：4,5,9,14,21,46,61,233    
   ```

JAVA实现代码：
```
import java.util.Arrays;

/**
 * Description: 基数排序
 * Author: zhangliangfu
 * Create on: 2019-08-22 13:22
 */
public class RadixSort {

    /**
     * 基数排序，LSD，低位优先
     * @param a
     */
    public static void radixSort(int[] a){
        //获得最大位数
        int max = a[0];
        for (int i = 0; i < a.length; i++) {
            if (a[i]>max){
                max = a[i];
            }
        }
        int d = 1;
        while ((max/=10)>0){
            d++;
        }
        //end
        //初始化接收容器(为了直观这里使用二维数组,可以使用List的动态链接属性避免开辟过大的二维数组浪费内存)
        int[][] container = new int[10][a.length];
        //循环所有位数
        for (int t = 0; t < d; t++) {
            //对元素进行除法和取余操作，获得相应位数上的值，并将之放入二维数组中对应的位置
            int[] ct = new int[10];
            for (int i = 0; i < a.length; i++) {
                int pow = (int) Math.pow(10, t);
                int num = a[i]/pow%10;
                container[num][ct[num]++] = a[i];
            }
            //用二维数组中的元素重建a[]
            int c = 0;
            for (int i = 0; i < 10; i++) {
                for (int j = 0; j < ct[i]; j++) {
                    a[c++] = container[i][j];
                }
            }
            //重建结束
        }
    }

    public static void main(String[] args) {
        int[] a = {9,5,14,4,21,46,233,61};
        radixSort(a);
        System.out.println(Arrays.toString(a));
    }
}
```