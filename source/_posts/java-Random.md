---
title: java-- Random
mathjax: false
date: 2018-10-10 09:51:36
categories:
    - java
    - Random
tags:
    - java
    - Random
---

#### 1.什么是Random类

此类的实例用于生成伪随机数。 

例如，以下代码使用户能够得到一个随机数： 

```
Random r = new Random(); 
int i = r.nextInt();
```

####2.Random使用步骤   

##### 查看类

* `java.util.Random` ：该类需要 import导入使后使用。

##### 查看构造方法 

* `public Random()` ：创建一个新的随机数生成器。

##### 查看成员方法 

* `public int nextInt(int n) `：返回一个伪随机数，范围在 0 （包括）和 指定值 n （不包括）之间的 

  int 值。 

##### 示例 1.生成随机数

```java
package com.zmjaction.demo01;
/*
Random类用来生成随机数字。使用起来也是三个步骤：

1. 导包
import java.util.Random;

2. 创建
Random r = new Random(); // 小括号当中留空即可

3. 使用
获取一个随机的int数字（范围是int所有范围，有正负两种）：int num = r.nextInt()
获取一个随机的int数字（参数代表了范围，左闭右开区间）：int num = r.nextInt(3)
实际上代表的含义是：[0,3)，也就是0~2
 */

import java.util.Random;

public class Demo01Random {
    public static void main(String[] args) {
        Random r = new Random();
        int num = r.nextInt(10);
        System.out.println(num);
    }
}

```

##### 示例 2

```java
package com.zmjaction.demo01;

import java.util.Random;

/*
题目要求：
根据int变量n的值，来获取随机数字，范围是[1,n]，可以取到1也可以取到n。

思路：
1. 定义一个int变量n，随意赋值
2. 要使用Random：三个步骤，导包、创建、使用
3. 如果写10，那么就是0~9，然而想要的是1~10，可以发现：整体+1即可。
4. 打印随机数字
 */
public class Demo02Random {

    public static void main(String[] args) {
        int n = 5;
        Random r = new Random();

        for (int i = 0; i < 100; i++) {
            // 本来范围是[0,n)，整体+1之后变成了[1,n+1)，也就是[1,n]
            int result = r.nextInt(n) + 1;
            System.out.println(result);
        }

    }

}
```

##### 示例3

```java
package com.zmjaction.demo01;

import java.util.Random;
import java.util.Scanner;

/*
题目：
用代码模拟猜数字的小游戏。

思路：
1. 首先需要产生一个随机数字，并且一旦产生不再变化。用Random的nextInt方法
2. 需要键盘输入，所以用到了Scanner
3. 获取键盘输入的数字，用Scanner当中的nextInt方法
4. 已经得到了两个数字，判断（if）一下：
    如果太大了，提示太大，并且重试；
    如果太小了，提示太小，并且重试；
    如果猜中了，游戏结束。
5. 重试就是再来一次，循环次数不确定，用while(true)。
 */
public class Demo03RandomGame {

    public static void main(String[] args) {
        Random r = new Random();
        int randomNum = r.nextInt(100) + 1; // [1,100]
        Scanner sc = new Scanner(System.in);

        while (true) {
            System.out.println("请输入你猜测的数字：");
            int guessNum = sc.nextInt(); // 键盘输入猜测的数字

            if (guessNum > randomNum) {
                System.out.println("太大了，请重试。");
            } else if (guessNum < randomNum) {
                System.out.println("太小了，请重试。");
            } else {
                System.out.println("恭喜你，猜中啦！");
                break; // 如果猜中，不再重试
            }
        }

        System.out.println("游戏结束。");
    }

}

```

