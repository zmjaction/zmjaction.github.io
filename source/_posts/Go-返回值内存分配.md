---
title: Go -- 返回值内存分配
mathjax: false
date: 2020-07-26 22:09:07
categories:
    - Go
    - defer
tags:
    - Go
    - defer
---
局部变量内存由所在的函数分配，但返回值内存却是由调用方（caller）提供，这种差别会导致某些行为上的差异

> defer 注册延迟调用，确保再函数结束前（ret）调用这些逻辑

注意，return语句执行顺序是：
* 1. 写返回值
* 2. 调用defer
* 3. 执行ret指令

```go  
func test() int {
	x := 0                 // x 内存由 test 栈帧提供。
    
	defer func() { 
        x = 100            // 闭包引用本地局部变量。
    }()   

	x = 200
	return x               // 将 200 写入返回值内存，然后才会执行 defer、ret。
}

func main() {
	println(test())        // 200
}

```
**执行顺序**：
* 1. local.x := 0
* 2. 注册defer调用，构成对本地变量的闭包引用，但是匿名函数未执行。
* 3. local.x = 200
* 4. 修改main.return_value = 200
* 5. 调用defer.fn 修改local.x =100,不影响main.return_value
* 6. ret 指令结束test函数执行

当返回值是命令方式时，行为差异导致输出结果不同
```go 
func test() (x int) {
	defer func() { 
        x = 100 
    }()                      // 引用返回值内存，修改有效。
    
	x = 200
	return
}

func main() {
	println(test())         // 100
}
```
执行顺序：
* 1. 注册defer调用，闭包引用x,注意命名返回值内存由调用方（main）提供
* 2. x= main.return_value =200 直接修改了返回值内存
* 3. return 未提供显示值，默认x
* 4. 调用defer函数，直接修改 x= main.return_value =100
* 5. ret指令结束函数执行

```go 

$ go tool objdump -s "main\.test" test
//用go tool objdump，可以看到任意函数的机器码、汇编指令、偏移
TEXT main.test(SB)  
  SUBQ $0x28, SP                                +----------+---------
  MOVQ BP, 0x20(SP)                             |          |  func1
  LEAQ 0x20(SP), BP                             +----------+---------
                                                | IP       |
  MOVUPS X0, 0x10(SP)                       0x0 +----------+--------- test.SP
  MOVB $0x0, 0xf(SP)                            | 返回值地址 | 
  MOVQ $0x0, 0x30(SP)      # 初始化返回值     0x8 +----------+       
                                                |          |
  LEAQ go.func.*+72(SB), AX                0x10 +----------+
  MOVQ AX, 0x18(SP)                             | 返回值地址 |
  LEAQ 0x30(SP), AX        # AX:返回值地址   0x18 +----------+  test
  MOVQ AX, 0x10(SP)        # 0x10(SP):AX        |          |
  MOVB $0x1, 0xf(SP)                       0x20 +----------+
  MOVQ $0xc8, 0x30(SP)     # 返回值 = 200        | BP       |
  MOVB $0x0, 0xf(SP)                       0x28 +----------+----------
  MOVQ 0x10(SP), AX        # AX:返回值地址        | IP       |
  MOVQ AX, 0(SP)           # 0x(SP):AX     0x30 +----------+---------- 
  CALL main.test.func1(SB)                      | 返回值内存 |
                                                +----------+  main
  MOVQ 0x20(SP), BP                             |          |
  ADDQ $0x28, SP                                +----------+
  RET                 

TEXT main.test.func1(SB)  # defer 注册的匿名函数。
  MOVQ 0x8(SP), AX        # 获取返回值地址
  MOVQ $0x64, 0(AX)       # 返回值 = 100
  RET 
```

源地址：https://www.yuque.com/docs/share/5a0474df-5bed-4aca-a1a2-82b58a16fc77