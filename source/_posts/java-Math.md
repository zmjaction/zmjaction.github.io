---
title: java-- Math类
mathjax: false
date: 2018-10-13 19:42:51
categories:
    - java
    - Math
tags:
    - java
    - Math

---

##### Math类

`java.lang.Math` 类包含用于执行基本数学运算的方法，如初等指数、对数、平方根和三角函数。类似这样的工具 

类，其所有方法均为静态方法，并且不会创建对象

##### 运算方法

* `public static double abs(double a)` ：返回 double 值的绝对值。
* `public static double ceil(double a)` ：返回大于等于参数的最小的整数。 
* `public static double floor(double a)` ：返回小于等于参数最大的整数。 
* `public static long round(double a)` ：返回最接近参数的 long。(相当于四舍五入方法) 

```java
public class demoMath {
    public static void main(String[] args) {
        // 获取绝对值
        System.out.println(Math.abs(3.14));

        // 向上取整
        System.out.println(Math.ceil(3.9));

        // 向下取整
        System.out.println(Math.floor(3.9));

        // 四舍五入
        System.out.println(Math.round(3.5));
    }
}
```

