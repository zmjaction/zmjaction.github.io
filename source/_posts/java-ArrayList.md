---
title: java-- ArrayList
mathjax: false
date: 2018-10-11 10:20:06
categories:
    - java
    - ArrayList
tags:
    - java
    - ArrayList
---

#### 1.什么要用ArrayList

```java
package com.zmjaction.demo01;
/*
数组有一个缺点：一旦创建，程序运行期间长度不可以发生改变。
 */
public class Students {
    private String name;
    private int age;

    public Students() {
    }

    public Students(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public static void main(String[] args) {
        // 首先创建一个长度为3的数组，里面用来存放Students类型的对象
        Students[] array = new Students[3];
        Students s1 = new Students("小明", 10);
        Students s2 = new Students("小李", 20);
        Students s3 = new Students("Tom", 30);
        array[0] = s1;
        array[1] = s2;
        array[2] = s3;
        System.out.println(array[0].getName());

    }
}
```

#### 2.ArrayList使用步骤

##### 查看类

* `java.util.ArrayList<E>`:该类需要 import导入使后使用

<E> ，表示一种指定的数据类型，叫做泛型。 E ，取自Element（元素）的首字母。在出现 E 的地方，我们使 

用一种引用数据类型将其替换即可，表示我们将存储哪种引用类型的元素。代码如下：

```java
ArrayList<String>，ArrayList<Student>
```

##### 查看构造方法

* `public ArrayList()` ：构造一个内容为空的集合

基本格式：

```
ArrayList<String> list = new ArrayList<String>();
```

在JDK 7后,右侧泛型的尖括号之内可以留空，但是<>仍然要写。简化格式： 

```
ArrayList<String> list = new ArrayList<>();
```

#### 4.常用的方法和遍历

对于元素的操作基本体现在-增，删，查

* `public boolean add(E e)` ：将指定的元素添加到此集合的尾部。 
* `public E remove(int index)` ：移除此集合中指定位置上的元素。返回被删除的元素。 
* `public E get(int index)` ：返回此集合中指定位置上的元素。返回获取的元素。 
* `public int size()` ：返回此集合中的元素数。遍历集合时，可以控制索引范围，防止越界。

```java
package com.zmjaction.demo01.d1;

import java.util.ArrayList;
/*
数组的长度不可以改变
但是ArrayList的长度是可以随意改变的
对于ArrayList来说，有一个尖括号<E>表示泛型
泛型：装在集合中的元素，统一是什么类型
注意：泛型只能是引用类型，不能是基本类型
引用类型：
基本类型：
注意：
对于ArrayList集合来说，直接打印得到的不是地址值，而是内容。
如果内容是空，得到的是空的中括号：[]
===============================================
 */
public class ArrayListDemo01 {
    public static void main(String[] args) {
        // 创建了一个ArrayList集合，集合名称是list，里边的元素是String类型的数据
        ArrayList<String> list = new ArrayList<>();
        // 想集合中添加一些元素
        list.add("xiaoming");
        list.add("xiaohong");
        list.add("xiaofeng");
        System.out.println(list);
        // 从集合中获取到元素：get 索引从0开始
        String name = list.get(2);
        System.out.println(name);
        // 从集合中删除元素，remove 索引值从0开始
        String rename = list.remove(2);
        System.out.println(rename);
        // 获取集合的长度，也就是元素的个数
        int size = list.size();
        System.out.println(size);
    }
}

```

#### 如何存基本数据类型数据
| 数据类型        | 大小（bit) | 范围                       | 默认值  | 包装类    |
| --------------- | ---------- | -------------------------- | ------- | --------- |
| byte(字节)      | 8          | -128 - 127                 | 0       | Byte      |
| short(短整型)   | 16         | -32768~32767               | 0       | Short     |
| int(整型)       | 32         | -2^31 ~ 2^31 -1            | 0       | Integer   |
| long(长整型)    | 64         | -2^63 ~ 2^63 -1            | 0L      | Long      |
| float(浮点型)   | 32         | 1.4013E-45~3.4028E+38      | 0.0f    | Float     |
| double(双精度)  | 64         | 4.9E-324~1.7977E+308       | 0.0d    | Double    |
| char(字符型)    | 16         | 0-65535 / `\u0000~ \uffff` | 'u0000' | Character |
| boolean(布尔型) | 1          | true/false                 | false   | Boolean   |

```java
package com.zmjaction.demo01.d1;

import java.util.ArrayList;
import java.util.Random;

public class ArrayListDemo02 {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();
        Random r = new Random();
        for (int i = 0; i < 10; i++) {
            int num = r.nextInt(30) + 1;
            list.add(num);
        }
        //便利集合
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }

    }
}

```





































