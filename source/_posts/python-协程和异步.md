---
title: python -- 协程和异步
date: 2019-07-05 00:10:10
categories:
    - python
tags:
    - python
    - Coroutine
    - 协程
---

#### 1. 并发、并行、同步、异步、阻塞、非阻塞

> 并发：一个时间段内，有几个程序在同一个 CPU 上运行，但是任意时刻只有一个程序在 CPU 上运行。
> 并行：在任意时刻点上，有多个程序同时运行在**多个 CPU **上。如果 CPU 有个四颗，那么并行最多只有四个。
> 基于以上，我们都说高并发，不说高并行。
> 同步：指代码调用 IO 操作时，必须等待 IO 操作完成才返回的调用方式。
> 异步：指代码调用 IO 操作时，不必等 IO 操作完成就返回的调用方式。
> 阻塞：指调用函数时，当前线程被挂起。
> 非阻塞：指调用函数时，当前线程不会被挂起，而是立即返回。
> 阻塞和非阻塞是说的函数调用的一种机制。

<!-- more -->

### C10k问题：

> 何如在一颗1Ghz CPU，2G内存，1gbps网络环境下，让单台服务器为1万个客户端提供FTP服务
>
> 处理用户的并发可以使用threading线程，模式来完成，但是一个线程只能处理一个socket,一个线程处理一个用户，几乎不可能完成上万用户的，如何在服务器上处理多个用户请求

### 2. IO 多路复用

### 2.1 Unix 下的五种 IO 模型

1. 阻塞式 IO
2. 非阻塞式 IO
3. IO 复用
4. 信号驱动式 IO
5. 异步 IO （POSIX 的 aio 系列函数）

### 2.2 阻塞式 IO



![](https://img-blog.csdnimg.cn/20181220170934897.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

> 在发起 IO 请求后，当前线程会阻塞，直到响应请求。当前线程阻塞后，就会造成 CPU 等待，造成 CPU 资源浪费。

### 2.3 非阻塞式 IO

![](https://img-blog.csdnimg.cn/20181220171045861.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

> 当发起 IO 请求后，需要不停的发起请求询问是否完成响应，通常这种询问是放在 while 循环里面完成的，但是，while 循环是非常耗费 CPU 的。而之前的阻塞式 IO 是不需要耗费 CPU，它只是当前线程被阻塞了而已。由此可知，并不是非阻塞式 IO 就比阻塞式 IO 好，实际开发中需要根据实际情况做具体分析。
> 阻塞和非阻塞 IO 方式都是调用系统级的recvfrom函数。同时，这两种模型都需要将系统地址空间中的数据报拷贝到用户地址空间。不懂的同学需要补充操作系统知识。

### 2.4 复用 IO

![](https://img-blog.csdnimg.cn/20181220170809848.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

> 复用 IO 模型克服了非阻塞式 IO 的缺点，避免了不停发请求造成的 CPU 资源浪费。这里调用的是系统级别的 select函数，等响应完成后通知发起请求的线程。select 是一个阻塞方法，在发起请求后阻塞当前线程。 select 和使用 while 循环方式的一个很大的不同点在于：select 可以监听多个请求线程句柄。一旦其中某个线程句柄状态发生变化，就可以立即处理这个响应完成的线程。
> 但是，这种模式依然需要将数据报从系统地址空间拷贝到用户地址空间，这也需要时间。
> 这种方式使用的非常广泛。

### 2.5 信号驱动式 IO

![](https://img-blog.csdnimg.cn/2018122017244488.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

> > 这种方式是基于信号处理的，目前使用很少。

### 2.6 异步 IO

![](https://img-blog.csdnimg.cn/20181220172710413.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

> 这种方式才是真正的异步 IO，但是这种方式并没有大规模应用，很多高并发框架也都没有采用这种方式，复用 IO 方式比较成熟，也比较稳定。并且，异步 IO 在实际使用过程中，aio_read 并没有发现有多大的提升。此外，这种方式编程难度比复用 IO 方式难。
> 这种方式省略了通知准备好数据报请求的时间。

### 2.7 select, poll, epoll 系统函数

![](https://img-blog.csdnimg.cn/20181220173540552.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/2018122017370667.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20181220173915308.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

> > poll 和 select 一样，都需要轮询。

![](https://img-blog.csdnimg.cn/20181220174055959.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

> epoll 只能在 Linux 中使用。epoll 使用红黑树这种数据结构进行查询，效率比较高。
> **但是，并不表示 epoll 一定就比 select 好。** 在并发高的情况下，连接活跃度不是很高， epoll比select好。并发性不高，同时连接很活跃， select比epoll好

### 3. 使用非阻塞 IO 方式建立 HTTP 请求

```python
#通过非阻塞io实现http请求

import socket
from urllib.parse import urlparse


#使用非阻塞io完成http请求

def get_url(url):
    #通过socket请求html
    url = urlparse(url)
    host = url.netloc
    path = url.path
    if path == "":
        path = "/"

    #建立socket连接
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 这一句非常重要，设置非阻塞方式，不管有没有数据准备好都是返回的
    client.setblocking(False)
    try:
        client.connect((host, 80)) #阻塞不会消耗cpu
    except BlockingIOError as e:
        pass

    #不停的询问连接是否建立好， 需要while循环不停的去检查状态
    #做计算任务或者再次发起其他的连接请求

    while True:
        try:
            client.send("GET {} HTTP/1.1\r\nHost:{}\r\nConnection:close\r\n\r\n".format(path, host).encode("utf8"))
            break
        except OSError as e:
            pass


    data = b""
    while True:
        try:
            d = client.recv(1024)
        except BlockingIOError as e:
            continue
        if d:
            data += d
        else:
            break

    data = data.decode("utf8")
    html_data = data.split("\r\n\r\n")[1]
    print(html_data)
    client.close()

if __name__ == "__main__":
    get_url("http://www.baidu.com")
```

### 4. 使用多路复用 IO 方式建立网络请求

>
>   下面这种方式的好处是并发性高，重点掌握。

```python
# 通过多路复用 IO 实现http请求
# 实现方式：select + 回调 + 事件循环
# 好处是：并发性高
# 使用单线程

import socket
from urllib.parse import urlparse
# 通常不实用  import select，而是使用 from selectors
# selectors 进行了封装。DefaultSelector 封装的更好用，调用 select 方法的时候，会根据平台选择使用 epoll 还是 select。避免了 epoll 在 Windows 下不能使用的情况。
# 在 Linux 下使用 epoll，在 Windows 下使用 select 方法
from selectors import DefaultSelector, EVENT_READ, EVENT_WRITE


selector = DefaultSelector()
# 使用select完成http请求
urls = []
stop = False


class Fetcher:
    def connected(self, key):
        # 在 send 之前，需要取消注册。
        # key.kd 就是 self.client.fileno() 的返回值
        selector.unregister(key.fd)
        # 在这里不需要再 try/catch，因为 connected 函数在调用的时候就已经表示时间就绪了
        self.client.send("GET {} HTTP/1.1\r\nHost:{}\r\nConnection:close\r\n\r\n".format(
            self.path, self.host).encode("utf8"))
        # 已经发送了数据，等到响应返回，故应该是读事件
        # 这里是 self.readable，不是 self.readable()
        selector.register(self.client.fileno(), EVENT_READ, self.readable)

    def readable(self, key):
        # 这里读数据没有放在 While True 中
        # readable 函数在每次可读的时候，会被自动调用，不再需要自己不停的读
        # 将读完的数据放到一个外部的变量中 self.data
        d = self.client.recv(1024)
        if d:
            self.data += d
        else:
            selector.unregister(key.fd)
            data = self.data.decode("utf8")
            html_data = data.split("\r\n\r\n")[1]
            print(html_data)
            self.client.close()
            urls.remove(self.spider_url)
            if not urls:
                global stop
                stop = True

    def get_url(self, url):
        self.spider_url = url
        url = urlparse(url)
        self.host = url.netloc
        self.path = url.path
        self.data = b""
        if self.path == "":
            self.path = "/"

        # 建立socket连接
        # 这里将 client 设置为 self，因为回调函数会用到
        self.client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.client.setblocking(False)

        try:
            self.client.connect((self.host, 80))
        except BlockingIOError as e:
            pass

        # 注册
        # 第一个参数：文件描述符；第二个参数：事件；第三个：回调函数，即当事件可写的时候，执行什么函数逻辑
        selector.register(self.client.fileno(), EVENT_WRITE, self.connected)

# loop 函数是实现的核心
# 回调是需要自己来做的，并不是操作系统调用回调函数
def loop():
    # 事件循环，不停的请求socket的状态并调用对应的回调函数
    # 事件循环这种模式在使用 IO 多路复用方式时会大量存在。
    # 1. select本身是不支持register模式。selector 对 select 进行了封装，故可以支持 register
    # 2. socket状态变化以后的回调是由程序员完成的
    while not stop:
        # 这里的 stop 是全局变量
        # selector.select() 在 Windows 下面调用的是 select 方法，注意，当传入的列表为空时，会报错
        # 这也是为啥设置了一个全局 stop 变量。注意和 readable() 中的代码结合起来看，需要仔细体会。为了解决这个问题，还设置了一个 urls = [] 和 self.spider_url = url
        ready = selector.select()
        for key, mask in ready:
            call_back = key.data
            call_back(key)
    # 回调+事件循环+select(poll\epoll)


if __name__ == "__main__":
    fetcher = Fetcher()
    import time
    start_time = time.time()
    for url in range(20):
        url = "http://shop.projectsedu.com/goods/{}/".format(url)
        urls.append(url)
        fetcher = Fetcher()
        fetcher.get_url(url)
    loop()
    print(time.time()-start_time)
```

# 5. 协程

## 5.1 回调之痛

> 上面的代码使用了回调，回调虽然有比较高的性能，但是有一些明显的缺点。缺点主要有以下三点：

1. 可读性差；
2. 共享状态管理困难；
3. 异常处理困难；

> 为了解决回调的缺点，解决的办法是：

1. 采用同步的方式去编写异步代码
2. 使用单线程去切换任务：
   1. 线程是由操作系统切换的，单线程切换意味着我们需要程序员自己去调度任务；
   2. 不再需要锁，并发性高，如果单线程内切换函数，性能远高于线程切换，并发性更高

### 5.2 生成器进阶

```python
协程，也被称作有多个入口的函数，或者可以暂停的函数（可以向暂停的地方传入值）。协程的实现方式：生成器。协程是函数级别的调用，由程序员决定自己调用逻辑。线程和进程是系统级别的调用。
生成器：

生成器不止可以产生值，还可以接收值
下面介绍三个方法，通过这生成器中的这三个方法实现协程。
```

### 5.2.1 send 方法

> send方法可以传递值进入生成器内部，同时还可以重启生成器执行到下一个yield位置。
> send 方法启用生成器时，需要先 send(None)，以使生成器启用。而 next 方法不需要。
> 除了 send 方法之外，生成器启用的另一个方式是 next 方法，next(gen)。

```python
def gen_func():
    # 下面这行代码有两个作用：
    # 1. 可以产出值， 2. 可以接收值(调用方传递进来的值)
    # 在运行 html = "bobby" 和 gen.send(html) 之后，会打印 bobby，表明可以接收外面传进来的值
    html = yield "http://projectsedu.com"
    print(html)
    return "bobby"

if __name__ == "__main__":
    gen = gen_func()
    # 在调用send发送非none值之前，我们必须启动一次生成器， 方式有两种1. gen.send(None), 2. next(gen)
    url = gen.send(None)
    # download url
    html = "bobby"
    print(gen.send(html))  # send方法可以传递值进入生成器内部，同时还可以重启生成器执行到下一个yield位置
    print(gen.send(html))
```

### 5.2.2 close 方法

```python
def gen_func():
    # 1. 可以产出值， 2. 可以接收值(调用方传递进来的值)
    try:
        yield "http://projectsedu.com"
    # GeneratorExit是继承自BaseException， 不是继承Exception
    # 捕捉 GeneratorExit 或者 BaseException 都会报错，捕捉 Exception 不会报错
    except GeneratorExit:
        pass
    yield 2
    yield 3
    return "bobby"


if __name__ == "__main__":
    gen = gen_func()
    print(next(gen))
    # 第一点：调用 close 方法后，如果后面的代码还有 yield 会抛出 GeneratorExit 异常。
    # 上面的例子中，后面有 yield 2 和 yield 3。如果注释掉这两行，close 是不会报错的。
    # 第二点：但是，close 之后，如果再执行 next(gen)，会抛出 StopIteration 异常。
    # 第三点：上面的例子，对 yield "http://projectsedu.com" 进行了 try/catch，会报 RuntimeError 异常。
    # 第三点：不要擅自进行 try/catch。如果 try/catch，需要在 catch 中抛出 raise StopIteration。
    gen.close()
    print("bobby")

```

### 5.2.3 throw 方法

```python
def gen_func():
    #1. 可以产出值， 2. 可以接收值(调用方传递进来的值)
    try:
        yield "http://projectsedu.com"
    except Exception as e:
        pass
    yield 2
    yield 3
    return "bobby"

if __name__ == "__main__":
    gen = gen_func()
    print(next(gen))
    # 下面这一句的异常是第 4 行的异常，返回值是 2
    gen.throw(Exception, "download error")
    # 打印值是 3
    print(next(gen))
    gen.throw(Exception, "download error")
```

### 5.3 yield from 语法

### 5.3.1 yield 和 yield from 的区别

> yield from 后面接一个 iterable 对象。yield from 不仅仅是不断的 yield iterable 中的值，还进行了很多额外的操作，参考源码即可。

```python
from itertools import chain

my_list = [1, 2, 3]
my_dict = {
    "url1": "http://www.baidu.com",
    "url2": "http://www.google.com"
}


# for value in chain(my_list, my_dict, range(5, 10)):
#     print(value)

# chain 内部的实现

# 分别使用yield / yield from
def my_chain(*args, **kwargs):
    for my_iterable in args:
        yield from my_iterable
        # for value in my_iterable:
        #     yield value


for value in my_chain(my_list, my_dict, range(5, 10)):
    print(value)

```

```python
def g1(iterable):
    yield iterable

def g2(iterable):
    yield from iterable

for value in g1(range(10)):
    print(value)
for value in g2(range(10)):
    print(value)

```

### 5.3.2 yield from 完成统计销量

```python
final_result = {}

def sales_sum(pro_name):
    total = 0
    nums = []
    while True:
        x = yield
        print(pro_name+"销量: ", x)
        if not x:
            break
        total += x
        nums.append(x)
    return total, nums

def middle(key):
    while True:
        # sales_sum 中的返回值是两个元素，故 final_result 中的 value 是 tuple
        final_result[key] = yield from sales_sum(key)
        print(key+"销量统计完成！！.")

def main():
    data_sets = {
        "bobby牌面膜": [1200, 1500, 3000],
        "bobby牌手机": [28,55,98,108 ],
        "bobby牌大衣": [280,560,778,70],
    }
    for key, data_set in data_sets.items():
        print("start key:", key)
        # 在 middle 方法中，调用了 yield from，从而将 yield from sales_sum(key) 的返回值和 当前的 main 函数建立了双向通道 
        m = middle(key)
        m.send(None) # 预激middle协程
        for value in data_set:
            # 不断地向 sales_sum 中发送值，值会被传到 x = yield 中（重点）
            m.send(value)   # 给协程传递每一组的值
        # 下面一行代码会导致 sales_sum 函数中 if not x: break，进而运行 return 语句，从而导致抛出 StopIteration
        m.send(None)
    print("final_result:", final_result)

if __name__ == '__main__':
    main()
```

> > 上面的例子，如果不使用 yield from，使用 yield 需要自己处理 StopIteration 异常。

```python
final_result = {}

def sales_sum(pro_name):
    total = 0
    nums = []
    while True:
        x = yield
        print(pro_name+"销量: ", x)
        if not x:
            break
        total += x
        nums.append(x)
    return total, nums

if __name__ == "__main__":
    my_gen = sales_sum("bobby牌手机")
    my_gen.send(None)
    my_gen.send(1200)
    my_gen.send(1500)
    my_gen.send(3000)
    try:
        # 下面一行代码会导致 sales_sum 函数中 if not x: break，进而运行 return 语句，从而导致抛出 StopIteration
        my_gen.send(None)
    except StopIteration as e:
        result = e.value
        print(result)
```

### 5.3.3 yield from 深入分析

> > 下面的代码非常有难度，只建议有非常扎实基础的人学习。

```python
# pep380

# 1. RESULT = yield from EXPR可以简化成下面这样
# 一些说明
"""
_i：子生成器，同时也是一个迭代器
_y：子生成器生产的值
_r：yield from 表达式最终的值
_s：调用方通过send()发送的值
_e：异常对象

"""

_i = iter(EXPR)      # EXPR是一个可迭代对象，_i其实是子生成器；
try:
    _y = next(_i)   # 预激子生成器，把产出的第一个值存在_y中；
except StopIteration as _e:
    _r = _e.value   # 如果抛出了`StopIteration`异常，那么就将异常对象的`value`属性保存到_r，这是最简单的情况的返回值；
else:
    while 1:    # 尝试执行这个循环，委托生成器会阻塞；
        _s = yield _y   # 生产子生成器的值，等待调用方`send()`值，发送过来的值将保存在_s中；
        try:
            _y = _i.send(_s)    # 转发_s，并且尝试向下执行；
        except StopIteration as _e:
            _r = _e.value       # 如果子生成器抛出异常，那么就获取异常对象的`value`属性存到_r，退出循环，恢复委托生成器的运行；
            break
RESULT = _r     # _r就是整个yield from表达式返回的值。

"""
1. 子生成器可能只是一个迭代器，并不是一个作为协程的生成器，所以它不支持.throw()和.close()方法；
2. 如果子生成器支持.throw()和.close()方法，但是在子生成器内部，这两个方法都会抛出异常；
3. 调用方让子生成器自己抛出异常
4. 当调用方使用next()或者.send(None)时，都要在子生成器上调用next()函数，当调用方使用.send()发送非 None 值时，才调用子生成器的.send()方法；
"""
_i = iter(EXPR)
try:
    _y = next(_i)
except StopIteration as _e:
    _r = _e.value
else:
    while 1:
        try:
            _s = yield _y
        except GeneratorExit as _e:
            try:
                _m = _i.close
            except AttributeError:
                pass
            else:
                _m()
            raise _e
        except BaseException as _e:
            _x = sys.exc_info()
            try:
                _m = _i.throw
            except AttributeError:
                raise _e
            else:
                try:
                    _y = _m(*_x)
                except StopIteration as _e:
                    _r = _e.value
                    break
        else:
            try:
                if _s is None:
                    _y = next(_i)
                else:
                    _y = _i.send(_s)
            except StopIteration as _e:
                _r = _e.value
                break
RESULT = _r

"""
看完代码，我们总结一下关键点：

1. 子生成器生产的值，都是直接传给调用方的；调用方通过.send()发送的值都是直接传递给子生成器的；如果发送的是 None，会调用子生成器的__next__()方法，如果不是 None，会调用子生成器的.send()方法；
2. 子生成器退出的时候，最后的return EXPR，会触发一个StopIteration(EXPR)异常；
3. yield from表达式的值，是子生成器终止时，传递给StopIteration异常的第一个参数；
4. 如果调用的时候出现StopIteration异常，委托生成器会恢复运行，同时其他的异常会向上 "冒泡"；
5. 传入委托生成器的异常里，除了GeneratorExit之外，其他的所有异常全部传递给子生成器的.throw()方法；如果调用.throw()的时候出现了StopIteration异常，那么就恢复委托生成器的运行，其他的异常全部向上 "冒泡"；
6. 如果在委托生成器上调用.close()或传入GeneratorExit异常，会调用子生成器的.close()方法，没有的话就不调用。如果在调用.close()的时候抛出了异常，那么就向上 "冒泡"，否则的话委托生成器会抛出GeneratorExit异常。

"""
```

### 5.4 async 和 wait 定义原生的协程

> **前面是使用 yield from 实现的协程，但是这种方式过于底层。python 提供了async和await关键词用于定义原生的协程。在实际开发中，建议使用这种方式。**前文的大幅章节主要用于帮助理解协程，以及协程怎么实现。

```python
#python为了将语义变得更加明确，就引入了async和await关键词用于定义原生的协程
async def downloader(url):
    return "bobby"

# 在 async 中不能使用 yield
async def download_url(url):
    #dosomethings
    html = await downloader(url)
    return html

if __name__ == "__main__":
    coro = download_url("http://www.imooc.com")
    # 不能这样调用 next(None)，应该使用下面这种方式，否则会报错
    coro.send(None
```

> await 关键字后面只能接 Awaitable 对象。await 后面的对象不能包含 yield 关键字，即下面代码的第一种实现。当使用了 types.coroutine 装饰后，就变成了 Awaitable 对象，也就不会报错了。

```python
def downloader(url):
    yield "bobby"

import types

@types.coroutine
def downloader(url):
    yield "bobby"
```

### 5.5 生成器是有状态的

```python
# 通过 inspect 模块获取生成器的状态
# 比如：inspect.getgeneratorstate(gen)
import inspect


def gen_func():
    # 下面这句有两层含义：
    # 第一：返回值给调用方； 第二： 调用方通过send方式返回值给gen
    value=yield 1
    return "bobby"


if __name__ == "__main__":
    gen = gen_func()
    print(inspect.getgeneratorstate(gen))
    next(gen)
    print(inspect.getgeneratorstate(gen))
    try:
        next(gen)
    except StopIteration:
        pass
    print(inspect.getgeneratorstate(gen))
```

### 6. 多路复用实现聊天群

```python
# Server 端
"""
server select
"""

import sys
import time
import socket
import select
import logging
from queue import Queue
import queue

g_select_timeout = 10

class Server(object):
    def __init__(self, host='0.0.0.0', port=3333, timeout=2, client_nums=10):
        self.__host = host
        self.__port = port
        self.__timeout = timeout
        self.__client_nums = client_nums
        self.__buffer_size = 1024

        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setblocking(False)
        self.server.settimeout(self.__timeout)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)  # keepalive
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  # 端口复用
        server_host = (self.__host, self.__port)
        try:
            self.server.bind(server_host)
            self.server.listen(self.__client_nums)
        except:
            raise

        self.inputs = [self.server]  # select 接收文件描述符列表
        self.outputs = []  # 输出文件描述符列表
        self.message_queues = {}  # 消息队列
        self.client_info = {}

    def run(self):
        while True:
            readable, writable, exceptional = select.select(self.inputs, self.outputs, self.inputs, g_select_timeout)
            if not (readable or writable or exceptional):
                continue

            for s in readable:
                if s is self.server:  # 是客户端连接
                    connection, client_address = s.accept()
                    # print "connection", connection
                    print("%s connect." % str(client_address))
                    connection.setblocking(0)  # 非阻塞
                    self.inputs.append(connection)  # 客户端添加到inputs
                    self.client_info[connection] = str(client_address)
                    self.message_queues[connection] = Queue()  # 每个客户端一个消息队列

                else:  # 是client, 数据发送过来
                    try:
                        data = s.recv(self.__buffer_size)
                    except:
                        err_msg = "Client Error!"
                        logging.error(err_msg)
                    if data:
                        # print data
                        data = "%s %s say: %s" % (time.strftime("%Y-%m-%d %H:%M:%S"), self.client_info[s], data)
                        self.message_queues[s].put(data)  # 队列添加消息

                        if s not in self.outputs:  # 要回复消息
                            self.outputs.append(s)
                    else:  # 客户端断开
                        # Interpret empty result as closed connection
                        print
                        "Client:%s Close." % str(self.client_info[s])
                        if s in self.outputs:
                            self.outputs.remove(s)
                        self.inputs.remove(s)
                        s.close()
                        del self.message_queues[s]
                        del self.client_info[s]

            for s in writable:  # outputs 有消息就要发出去了
                try:
                    next_msg = self.message_queues[s].get_nowait()  # 非阻塞获取
                except queue.Empty:
                    err_msg = "Output Queue is Empty!"
                    # g_logFd.writeFormatMsg(g_logFd.LEVEL_INFO, err_msg)
                    self.outputs.remove(s)
                except Exception as e:  # 发送的时候客户端关闭了则会出现writable和readable同时有数据，会出现message_queues的keyerror
                    err_msg = "Send Data Error! ErrMsg:%s" % str(e)
                    logging.error(err_msg)
                    if s in self.outputs:
                        self.outputs.remove(s)
                else:
                    for cli in self.client_info:  # 发送给其他客户端
                        if cli is not s:
                            try:
                                cli.sendall(next_msg.encode("utf8"))
                            except Exception as e:  # 发送失败就关掉
                                err_msg = "Send Data to %s  Error! ErrMsg:%s" % (str(self.client_info[cli]), str(e))
                                logging.error(err_msg)
                                print
                                "Client: %s Close Error." % str(self.client_info[cli])
                                if cli in self.inputs:
                                    self.inputs.remove(cli)
                                    cli.close()
                                if cli in self.outputs:
                                    self.outputs.remove(s)
                                if cli in self.message_queues:
                                    del self.message_queues[s]
                                del self.client_info[cli]

            for s in exceptional:
                logging.error("Client:%s Close Error." % str(self.client_info[cli]))
                if s in self.inputs:
                    self.inputs.remove(s)
                    s.close()
                if s in self.outputs:
                    self.outputs.remove(s)
                if s in self.message_queues:
                    del self.message_queues[s]
                del self.client_info[s]

if "__main__" == __name__:
    Server().run()
```

```python
# client 端

#!/usr/local/bin/python
# *-* coding:utf-8 -*-

"""
client.py
"""

import sys
import time
import socket
import threading

class Client(object):
    def __init__(self, host, port=3333, timeout=1, reconnect=2):
        self.__host = host
        self.__port = port
        self.__timeout = timeout
        self.__buffer_size = 1024
        self.__flag = 1
        self.client = None
        self.__lock = threading.Lock()

    @property
    def flag(self):
        return self.__flag

    @flag.setter
    def flag(self, new_num):
        self.__flag = new_num

    def __connect(self):
        client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # client.bind(('0.0.0.0', 12345,))
        client.setblocking(True)
        client.settimeout(self.__timeout)
        client.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  # 端口复用
        server_host = (self.__host, self.__port)
        try:
            client.connect(server_host)
        except:
            raise
        return client

    def send_msg(self):
        if not self.client:
            return
        while True:
            time.sleep(0.1)
            # data = raw_input()
            data = sys.stdin.readline().strip()
            if "exit" == data.lower():
                with self.__lock:
                    self.flag = 0
                break
            self.client.sendall(data.encode("utf8"))
        return

    def recv_msg(self):
        if not self.client:
            return
        while True:
            data = None
            with self.__lock:
                if not self.flag:
                    print('ByeBye~~')
                    break
            try:
                data = self.client.recv(self.__buffer_size)
            except socket.timeout:
                continue
            except:
                raise
            if data:
                print("%s\n" % data)
            time.sleep(0.1)
        return

    def run(self):
        self.client = self.__connect()
        send_proc = threading.Thread(target=self.send_msg)
        recv_proc = threading.Thread(target=self.recv_msg)
        recv_proc.start()
        send_proc.start()
        recv_proc.join()
        send_proc.join()
        self.client.close()

if "__main__" == __name__:
    Client('localhost').run()

# 运行方式：
# 1. 启动server
# python server.py

# 2. 启动client1
# python client.py

# 启动client2
# python client.py

# 在client1的console中输入任何字符串，client2中立马就可以收到
```

