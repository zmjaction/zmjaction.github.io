---
title: strace -- 跟踪器
date: 2018-10-09 01:48:40
categories:
    - Linux
    - strace
tags:
    - Linux
    - strace
---

#### 什么是strace

> strace是一个可用于诊断，调试，用户控件跟踪器，监控用户空间进程和内核的交互，如系统调用，信号传递，进程状态变更

<!-- more -->

#### strace常用选项：

从一个示例命令：

```bash
strace -tt -T -v -f -e trac=file -o /data/log/strace.log -s 1024 -p 23489
```

>-tt 在每行输出的前面，显示毫秒级别的时间
>
>-T 显示每次系统调用所花费的时间
>
>-v 对于某些相关调用，把完整的环境变量，文件stat结构打印出来
>
>-f 跟踪目标进程，以及目标进程穿件的所有子进程
>
>-e 控制要跟踪的时间和跟踪行为，如指定要跟踪的系统调用名称
>
>-o 把strace输出单独写到指定文件
>
>-s当系统调用的某个参数是字符串时，最多输出指定长度的内容，默认是32字节
>
>-p指定要跟踪的进程pid，要同时跟踪多个pid，重复多次-p选项即可

####系统调用函数分类说明

##### 进程管理



函数名 | 说明 
-|-
fork | 创建一个新进程 
clone | 按指定条件创建子进程 
execve | 运行可执行文件 
pause | 进程将除域阻塞状态 
wait | 等待子进程终止 
waitpid | 等待指定子进程终止 

##### 文件和设备访问

函数名 | 说明 
-|-
open | 打开文件 
creat | 创建新文件 
close | 关闭文件描述字 
read | 读文件 
write | 写文件 
pread | 对文件随机读 
pwirte | 对文件随机写 
poll | I/O多路转换 
truncate | 截断文件 

##### 文件系统操作
函数名 | 说明 
-|-
access | 确定文件的可存取 
chmod |  
chown | 改变文件的主，组 
chroot | 改变根目录 
stat | 获取文件状态信息 
lstat | 参见stat 
getdents | 读取目录项 
mkdir | 创建目录 
link | 创建链接 
##### 内存管路
函数名 | 说明 
-|-
mmap | 映射虚拟内存页 
sync | 将内存缓冲区数据写回硬盘 

##### socket控制
函数名 | 说明 
-|-
socketall | socket系统调用 
socket | 建立socket 
bind | 绑定socket端口 
connect | 链接远程主机 
send | 通过socket发送信息 
sendto | 发送UDP信息 
sendmsg | 参见send 
listen | 监听socket端口 
select | 对多路同步I/O进行轮询 

示例：

```
strace -ff -o out nc -l 8080
# -ff 抓取nc -l 8080 所有进程线程的对内核的调用
会产生一个nc -l 8080 进程所对应进程id的文件
如：out.92345
```

```
	无法链接到服务器
	ping www.baidu.com
	strace -e connect，select，poll，sendto www.baidu.com
```

