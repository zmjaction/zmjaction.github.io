---
title: java-- Arrays类
mathjax: false
date: 2018-10-13 19:39:43
categories:
    - java
    - Arrays类
tags:
    - java
    - Arrays

---

##### Arrays类

* `java.util.Arrays` 此类包含用来操作数组的各种方法，比如排序和搜索等，所有方法均为静态方法

##### 方法：

* `public static String toString(int[] a)` ：返回指定数组内容的字符串表示形式。 
* `public static void sort(int[] a) `：对指定的 int 型数组按数字升序进行排序。 

```java
import java.util.Arrays;

public class demotestArrays {
    public static void main(String[] args) {
        int[] intArrays = {10, 50, 30 ,20};
        // int[] 转换为str
        String intstr = Arrays.toString(intArrays);
        System.out.println(intstr);
        // sort 排序
        int[] array1 = {9,20,4,3,5,8,10,1};
        Arrays.sort(array1);
        System.out.println(Arrays.toString(array1));
        // sort 字符串排序
        String[] array2 = {"bb", "ccc","aaa","sgfagf"};
        Arrays.sort(array2);
        System.out.println(Arrays.toString(array2));
        // 

    }
}
```

```java
public class demo02convert {
    public static void main(String[] args) {
        //将一个随机字符串中的所有字符升序排列，并倒序打印
        String str1 = "sdsa34snsth92o3afahuoh4qd";
        char[] charstr1 = str1.toCharArray();
        System.out.println(charstr1);
        Arrays.sort(charstr1);
        for (int i = charstr1.length-1; i >= 0; i--) {
            System.out.println(charstr1[i]);
        }
    }
}
```

