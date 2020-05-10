---
title: nc 命令使用
date: 2020-05-10 03:30:40
categories:
    - Linux
    - nc
tags:
    - Linux
    - nc
---

什么是nc

> nc是netcat简称
>
> 1.实现任意TCP/UDP端口的监听
>
> 2.与telnet类似
>
> 3.端口的扫描
>
> 4.机器之间传输文件
>
> 5.机器之间网络测速 

**语法格式：**nc [参数]

```bash
-l: 作为监听，开启一个port监听用户的联机
-u: 不适用tcp而是使用udp作为连接的数据包状态
```

<!-- more -->

##### 安装

```bash
yum install nc
```



参考实例：

```bash
# 与telnet类似
nc localhost 25
```

```bash
# 建立两个联机来传讯
# 终端1中输入：监听tcp端口
nc -l localhost 8080
# 终端2中输入
nc localhost 8080
# 可以进行交互了
```

```bash
扫描tcp端口

nc -vz -w  192.168.1.105 9999
```

```bash
接收文件
# 终端1 192.168.1.105 file.txt 接收的文件名
nc -l 9995 >file.txt
# 终端2 deployment.yml需要发送的文件
nc 192.168.1.105 9995 < deployment.yml
```

```bash
测速
需要安装yum install -y dstat
# 终端1（192.168.1.105）  
nc -l 9991 >/dev/null
# 终端2 （192.168.1.106）
nc 192.168.1.105 9991 </dev/zero
# 终端3（192.168.1.105）
dstat
```



