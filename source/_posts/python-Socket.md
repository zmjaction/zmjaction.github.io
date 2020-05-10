---
title: python -- socket编程
date: 2019-07-03 02:30:23
categories:
    - python
tags:
    - python
    - socket
---


  ### 1. Socket 编程简介

![](https://img-blog.csdnimg.cn/20181109233232849.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

> 注意，**Socket 编程与 Http 请求不同，Socket 编程当连接完成后，就可以一直给另一方发送数据，只要连接没有断开，就可以一直发送数据。而 Http 请求是连接、发送数据、断开。每次发送数据都需要重复上面的过程，每次都需要连接。**这是非常重要的一点。

<!-- more -->

### 2. 基于 Socket 的简单聊天程序

### 2.1 服务器端

```python
import socket
import threading

# socket.AF_INET 可以理解成 IPv4
# socket.AF_INET6 可以理解成 IPv6
# socket.SOCK_STREAM TCP 通信
# socket.SOCK_DGRAM UDP 通信
server = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
# 注意，必须传入一个 tuple
# 这儿不能传入127.0.0.1，要不然就访问不到了
server.bind(('0.0.0.0', 8000))
server.listen()

#获取从客户端发送的数据
#一次获取1k的数据
def handle_sock(sock, addr):
    while True:
        data = sock.recv(1024)
        data = data.decode("utf8")
        if data.lower() == 'exit' or data.lower() == 'bye':
            break
        # 获取一个服务器端的输入，发送给客户端
        re_data = input()
        sock.send(re_data.encode("utf8"))
    sock.close()

while True:
    sock, addr = server.accept()

    # 用线程去处理新接收的连接(用户)，这样就可以同时接收多个客服端的请求
    # target 传的是函数的名称，不是函数的调用
    client_thread = threading.Thread(target=handle_sock, args=(sock, addr))
    client_thread.start()
```

### 2.2 客户端

```python
import socket
client = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
client.connect(('127.0.0.1', 8000))
while True:
    re_data = input()
    client.send(re_data.encode("utf8"))
    data = client.recv(1024)
    print(data.decode("utf8"))
```

### 3. 使用 Socket 模拟 Http 请求

```
# requests -> urlib -> socket
import socket
from urllib.parse import urlparse


def get_url(url):
    # 通过socket请求html
    url = urlparse(url)
    # 获取主机名
    host = url.netloc
    # 请求路径
    path = url.path
    # 当请求为空时，特殊处理
    if path == "":
        path = "/"

    # 建立socket连接
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # client.setblocking(False)
    client.connect((host, 80))  # 阻塞不会消耗cpu

    # 不停的询问连接是否建立好， 需要while循环不停的去检查状态
    # 做计算任务或者再次发起其他的连接请求

    # 注意这儿的编码
    client.send(
        "GET {} HTTP/1.1\r\nHost:{}\r\nConnection:close\r\n\r\n".format(path, host).encode("utf8"))

    # 注意这儿是 byte 类型
    data = b""
    while True:
        d = client.recv(1024)
        if d:
            data += d
        else:
            break

    data = data.decode("utf8")
    # 只保留 HTML 内容，去除头部内容
    html_data = data.split("\r\n\r\n")[1]
    print(html_data)
    client.close()


if __name__ == "__main__":
    import time
    start_time = time.time()
    for url in range(20):
        url = "http://shop.projectsedu.com/goods/{}/".format(url)
        get_url(url)
    print(time.time()-start_time)

```