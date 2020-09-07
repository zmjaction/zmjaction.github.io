
---
title: new和make的区别
mathjax: false
date: 2018-07-21 03:21:51
categories:
    - go
tags:
    - go
    - new
    - make
---

#### new函数

new函数的作用：

* 分配内存
* 设置零值
* 返回指针
<!-- more -->
例如：

```go
package main

import "fmt"

type Students struct {
	name string
	age int
}

func main() {
	// new 一个内建类型
	num := new(int)
	fmt.Println(*num) // 打印零值：0
	// new 一个定义类型
	stu := new(Students)
	stu.name = "TOM"
	stu.age = 19
	fmt.Println(*stu)
}
```

#### make 函数

make函数：

* 为slice，map，chan类型分配内存，初始化对象
* 返回类型的本身而不是指针，依赖于具体传入的类型，（slice，map，chan 本身就是引用类型，不必返回他们的指针）

```go
//切片
a := make([]int, 2, 10)  

// 字典
b := make(map[string]int)

// 通道
c := make(chan int, 10)
```

