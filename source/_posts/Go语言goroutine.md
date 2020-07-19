---
title: go -- goroutine--示例解析
mathjax: false
date: 2018-07-01 10:10:09
categories:
    - go
    - goroutine
tags:
    - go
    - goroutine
---

简单的goroutine示例：

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	var a [10]int
	for i := 0; i < 10; i++ {
		go func(i int) {
			for  {
				fmt.Printf("Hello goroutine %d\n", i) // i/O操作
			}
		}(i)
	}
	time.Sleep(time.Microsecond)
}

```
* 上边的代码如果不加go执行的话，只会在第一次i=0的时候无限循环执行`fmt.Printf("Hello goroutine %d\n", i)`
* main()也是一个goroutine，主goroutine已经结束， 子goroutine还没有执行，所以需要加上`time.Sleep(time.Microsecond)`

#### go开启的其实不是线程，而是协程
* 轻量级的协程
* **非抢占式** 多任务处理，由协程主动交出控制权
* 而线程再任何时候都可以被系统切换，所有线程又称之为抢占式多任务处理，哪怕程序执行到一半，都可以被系统切换
* 编译器，解析器或者虚拟机层面的多任务
* 协程可以看作是编译器层面的多任务而不是操作系统的多任务 操作系统只有线程没有协程
* go语言在执行goroutine时 编译器会有一个调度器为他调度协程 操作系统有调度器 go语言有自己的调度器
* 多个协程可能再一个或多个线程上运行

#### 非抢占式
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	var a [10]int
	for i := 0; i < 10; i++ {
		go func(i int) {
			for  {
				a[i]++
			}
		}(i)
	}
	time.Sleep(time.Microsecond)
	fmt.Println(a)
}
```
执行发现程序 夯住了，查看cpu占用，发现已经快达到100%了
产生的原因：
    * a[i]++ 无法主动交出控制权
    * main函数事主goroutine，由于 go func 没有交出控制权，导致main一直等待

从之前的代码可以看到，io操作时可以自动交出控制权的，如何手动交出控制权？可以使用`runtime.Gosched()`。一般情况下不会用到`runtime.Gosched()`
```go
func main() {
	var a [10]int
	for i := 0; i < 10; i++ {
		go func(i int) {
			for  {
				a[i]++
				runtime.Gosched()
			}
		}(i)
	}
	time.Sleep(time.Microsecond)
	fmt.Println(a)
}
```
#### go func(){}() 中不传入变量可不可以？
```go
// 通过执行发现 报错了
func main() {
	var a [10]int
	for i := 0; i < 10; i++ {
		go func() {
			for  {
				a[i]++
				runtime.Gosched()
			}
		}()
	}
	time.Sleep(time.Microsecond)
	fmt.Println(a)
}
```
通过执行发现报错了，但是为什么报错呢？
* go run -race goroutine.go 看下究竟发生了什么
* 从上边的执行命令,可以看出，出错的原因是因为如果不把i作为参数传给func，就不会形成闭包，在最后一次执行for的时候i会是10，导致a[10]超出范围

#### go语言调度器
在go语言里，具体每个协程如何分配到线程里，都是交给调度器来分配，我们不用操心。
#### goroutine的定义
* 直接在函数前加入go
* 不需要再定义时区分是否是异步函数
* 调度器在合适的点进行切换
* 使用-race检测数据访问冲突
#### goroutine的切换点
下面这些点可以参考，但是由于我们无法100%控制，所以只能是参考。
* I/O select
* channel
* 等待锁
* 函数调用（有时）
* runtime.Gosched()
