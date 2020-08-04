---
title: go -- 语言Channel
date: 2018-07-12 22:09:21
mathjax: false
categories:
    - go
    - Channel
tags:
    - go
    - Channel
---
#### Channel 
再使用goroutine的时候提到过多个goroutine会产生竞争关系。可以通过原子函数和互斥锁来解决竞争关系，更推荐使用channel
#### 定义channel
```go
//无缓冲的整型通道
unbuffered := make(chan int)
//有缓冲的字符串通道
buffered := make(chan string, 10)
```
#### 从channel里发送数据
```go
//通过通道发送一个字符串 
buffered <- 1
//从通道接收一个字符串 
value := <- buffered
```
示例1：
```go
package main

import "fmt"

func main() {
	c := make(chan int)
	go func(c chan int) {
		for {
			n := <- c
			fmt.Println(n)
		}
	}(c)
	c <- 1
	c <- 2
}
```
示例2：
```go
package main

import "fmt"

func main() {
	chanDemo()
}

func chanDemo() {
	// create channel slice
	var channels [10]chan int
	for i := 0; i < 10; i++ {
		//channels[i] = make(chan int)
		//go worker(i, channels[i])
		channels[i] = Createworker(i)
	}
	for i := 0; i <10; i++ {

		channels[i] <- 'a' + i
	}
	for i := 0; i <10; i++ {
		channels[i] <- 'A' + i
	}
}

func worker(id int, c chan int) {
	for  {
		fmt.Printf("Worker %d received %d\n",
			id, <-c)
	}
}
func Createworker(id int) chan int{
	c := make(chan int)
	go func() {
		for  {
			fmt.Printf("worker %d received %c\n", id ,<-c)
		}
	}()
	return c
}

```

#### 有缓冲区的channel
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	c := make(chan int,3)
	go worker(c)
	//for i := 0; i < 10; i++ {
	//	c <- 'a' +i
	//}
	c <- 'a'
	c <- 'b'
	c <- 'c'
	// 设置缓冲区，不读取数据也不会报错
	time.Sleep(time.Millisecond)
}

func worker(c chan int) {
	for  {
		fmt.Printf("worker received %c\n", <-c)
	}
}
```

// close() 关闭缓冲区
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	c := make(chan int,3)
	// 设置缓冲区，不读取数据也不会报错
	go worker(c)
	//for i := 0; i < 10; i++ {
	//	c <- 'a' +i
	//}
	c <- 'a'
	c <- 'b'
	c <- 'c'
	c <- 'd'
	close(c) // 关闭channel ,会继续发送该channel类型的零值数据
	time.Sleep(time.Millisecond)
}

func worker(c chan int) {
	//for  {
	//	n, ok := <- c
	//	if !ok{
	//		break
	//	}
	//	fmt.Printf("worker received %c\n", n)
	//}
	// 简单写法
	for n := range c{
		fmt.Printf("worker received %c\n", n)
	}
}
```
#### 如何等待Goroutine
在之前的代码里面都有使用time.Sleep(time.Millisecond)来等待Channel的结束。但是这种方法的危险肉眼可见。所以我们需要修改一下。
修改点：
* 在执行打印的时候，不仅接受一个Channel的int，同时在打印完成后，传出一个done
* 由于有一对chan，将这一对chan提出来做一个struct
* 在Createworker的时候用worker构造体传值
* 同时在创建10个worker的时候也用构造体创建构造体的slice
* 在向内部传值后，同时加入接受done的处理

```go
package main

import "fmt"

type worker struct {
	in chan int
	done chan bool
}
func main() {
	fmt.Println("channel as first_class citizen")
	chanDemo()
}

func chanDemo() {
	var workers [10]worker
	for i := 0; i < 10; i++ {
		workers[i] = createWorker(i)
	}
	for i := 0; i < 10; i++ {
		workers[i].in <- 'a' +i
	}
	//for i := 0; i < 10; i++ {
	//	<- workers[i].done
	//}
	for i := 0; i < 10; i++ {
		workers[i].in <- 'A' + i
	}
	for i := 0; i < 10; i++ {
		<- workers[i].done
		<- workers[i].done
	}

}

func createWorker(i int) worker {
	w := worker{
		in: make(chan int),
		done: make(chan bool),
	}
	go doworker(i , w.in, w.done)
	return w
}

func doworker(i int, c chan int, done chan bool) {
	for n := range c {
		fmt.Printf("Worker %d received %c\n", i, n)
		//done <- true
		go func() {
			done <- true
		}()
	}

}
```
#### 使用sync.WaitGroup 来管理goroutine结束
WaitGroup的用法：
* 定义WaitGroup本身：var wg sync.WaitGroup
* 定义需要等待的goroutine数量：wg.Add(20)
* 等待：wg.Wait()
* 在goroutine执行完成后done：w.done()

```go
package main

import (
	"fmt"
	"sync"
)

type worker struct {
	in chan int
	//done chan bool
	done func()
}
func main() {
	fmt.Println("channel as first_class citizen")
	chanDemo()
}

func chanDemo() {
	var wg sync.WaitGroup
	var workers [10]worker
	for i := 0; i < 10; i++ {
		workers[i] = createWorker(i, &wg)
	}
	wg.Add(20)
	for i := 0; i < 10; i++ {
		workers[i].in <- 'a' +i
	}
	//for i := 0; i < 10; i++ {
	//	<- workers[i].done
	//}
	for i := 0; i < 10; i++ {
		workers[i].in <- 'A' + i
	}
	//for i := 0; i < 10; i++ {
	//	<- workers[i].done
	//	<- workers[i].done
	//}
	wg.Wait()

}
func createWorker(i int, wg *sync.WaitGroup) worker {
	w := worker{
		in: make(chan int),
		//done: make(chan bool),
		done: func() {
			wg.Done()
		},
	}
	go doworker(i ,w)
	return w
}

func doworker(i int, w worker) {
	for n := range w.in {
		fmt.Printf("Worker %d received %c\n", i, n)
		//done <- true
		//go func() {
		//	done <- true
		//}()
		w.done()
	}

}
```
> Don't communicate by sharing memory, share memory by communicating.
#### 不要通过共享内存进行通信，要通过通信来共享内存
比如我们判断前后程序是否处理是否完成的时候经常设置一个flag变量，然后通过检测这个变量来进行判断，这就是所谓通过共享内存进行通信。但是go语言更提倡通过使用Channel，通过通信来共享内存。

```go
//简单示例
package main

import (
	"fmt"
	"math/rand"
	"time"
)
func main() {
	var c1, c2 = generator(), generator()
	for {
		select {
		case n:= <- c1:
			fmt.Println(n)
		case n := <- c2:
			fmt.Println(n)
		}
	}
	
}
func generator() chan int{
	out := make(chan int)
	go func() {
		i := 0
		for  {
			time.Sleep(time.Duration(rand.Intn(1500))*time.Millisecond)
			out <- i
			i ++

		}
	}()
	return out
}
```
```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func generator() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(
				time.Duration(rand.Intn(1500)) *
					time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}

func worker(id int, c chan int) {
	for n := range c {
		fmt.Printf("Worker %d received %c\n",
			id, n)
	}
}

func createWorker(id int) chan<- int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func main() {
	worker := createWorker(0)
	var c1, c2 = generator(), generator()
	n := 0
	hasValue := false
	for {
		// nil chan永远不会被select到
		var activeworker chan<- int 
		if hasValue {
			activeworker = worker
		}
		select {
		case n = <-c1:
			fmt.Println("Received from c1: ", n)
			hasValue = true
		case n = <-c2:
			fmt.Println("Received from c2: ", n)
			hasValue = true
		case activeworker <- n:
			hasValue = false
		}
	}
}
```



