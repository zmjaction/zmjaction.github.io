---
title: python -- 迭代器和生成器
date: 2019-07-01 01:10:23
categories:
    - python
tags:
    - python
    - iterator
    - yield
---

### 1. 迭代器 Iterator 和可迭代的 Iterable

> 迭代器是访问集合内元素的一种方式， 一般用来遍历数据。迭代器和以下标的访问方式不一样， 迭代器是不能返回的, 迭代器提供了一种惰性方式数据的方式。
> 可迭代的 Iterable 表示这个对象是可以迭代的。

```python
from collections.abc import Iterable, Iterator
a = [1,2]
iter_rator = iter(a)
print (isinstance(a, Iterable))
print (isinstance(iter_rator, Iterator))
```

<!-- more -->

### 2. 迭代器

### 2.1 第一个例子

> 使用 Python 内置的 iter 函数获得迭代器

```python
from collections.abc import Iterator

class Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list
	
	# 如果重写这个函数，需要返回迭代器，否则会报错
	# 优先调用 __iter__ 方法
    def __iter__(self):
        return MyIterator(self.employee)

	# 如果没有 __iter__ 方法，才会调用这个方法
    def __getitem__(self, item):
        return self.employee[item]

if __name__ == "__main__":
    company = Company(["tom", "bob", "jane"])
    
    my_itor = iter(company)
```

### 2.2 自定义迭代器

```python
from collections.abc import Iterator

class Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list

    def __iter__(self):
        return MyIterator(self.employee)

    def __getitem__(self, item):
        return self.employee[item]

# 继承默认的 Iterator
class MyIterator(Iterator):
    def __init__(self, employee_list):
        self.iter_list = employee_list
        self.index = 0

	# 迭代器是不支持切片的，需要自己维护索引，进而控制返回的数据
    def __next__(self):
        #真正返回迭代值的逻辑
        try:
            word = self.iter_list[self.index]
        except IndexError:
            raise StopIteration
        self.index += 1
        return word

if __name__ == "__main__":
    company = Company(["tom", "bob", "jane"])
    
    # 结束迭代的时候会返回 StopIteration，for 语句接收到这个 except，进而结束循环
    for item in company:
        print(item)
	# 上面的 for 语句实现的功能可以用如下代码替代
	while True:
        try:
            print (next(my_itor))
        except StopIteration:
            pass
```

### 3. 生成器

> 函数里只要有yield关键字就是生成器函数。

### 3.1 一个例子

```python
def fib(index):
    if index <= 2:
        return 1
    else:
        return fib(index-1) + fib(index-2)
print(fib(2),8888)
----------------------------------------------
def fib2(index):
    re_list = [] # 几十万上百万比较消耗内存
    n,a,b = 0,0,1
    while n<index:
        re_list.append(b)
        a,b = b, a+b
        n += 1
    return re_list
----------------------------------------------
def gen_fib(index):
    n,a,b = 0,0,1
    while n<index:
        yield b
        a,b = b, a+b
        n += 1

for data in gen_fib(10):
    print (data)
-------------------------------------------------
```

### 3.2 读取只有单行的大文件

> 假设有个文件有 500G，并且只有一行，使用通常的方法都无法满足需求，我们使用每次读取 read(4096) 个字符，再加上 yield 方式完成需求。

```python
#500G, 特殊 一行
def myreadlines(f, newline):
  buf = "" # 可以理解为缓存，存储已经读出的数据量
  while True:
    while newline in buf: # 查看缓存中是否有{|}分隔符
      pos = buf.index(newline)
      yield buf[:pos]
      buf = buf[pos + len(newline):]
    chunk = f.read(4096) # 刚开始时候是空字符串，先读4096个字符

    if not chunk:
      # 处理边缘条件
      #说明已经读到了文件结尾
      yield buf
      break
    buf += chunk

with open("input.txt") as f:
    for line in myreadlines(f, "{|}"):
        print (line)


```

