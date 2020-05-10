---
title: python -- 序列类
date: 2019-06-12 11:11:23
categories:
    - python
tags:
    - python
    - 序列类
---

python 序列类型

> 序列类型主要可分为以下几个类别：

1. 容器序列：list、tuple、deque
2. 扁平序列：str、bytes、bytearray、array.array
3. 可变序列：list， deque，bytearray、array
4. 不可变序列：str、tuple、bytes

<!-- more -->

### 1.bisect模块

> bisect 模块常常用来维护一个已排序的升序序列。

```python
import bisect
from collections import deque

#用来处理已排序的序列，用来维持已排序的序列， 升序
#二分查找
inter_list = deque()
bisect.insort(inter_list, 3)
bisect.insort(inter_list, 2)
bisect.insort(inter_list, 5)
bisect.insort(inter_list, 1)
bisect.insort(inter_list, 6)

print(bisect.bisect_left(inter_list, 3))
#学习成绩
print(inter_list)

```

### 2.array与list的不同点

1.array只能存在指定的数据类型，list可以存放各种数据类型

2.array性能高

```python
import array

my_array = array.array("i")
my_array.append(1)
my_array.append("abc")

```

### 3.切片操作

```python
#模式[start:end:step]
"""
    其中，第一个数字start表示切片开始位置，默认为0；
    第二个数字end表示切片截止（但不包含）位置（默认为列表长度）；
    第三个数字step表示切片的步长（默认为1）。
    当start为0时可以省略，当end为列表长度时可以省略，
    当step为1时可以省略，并且省略步长时可以同时省略最后一个冒号。
    另外，当step为负整数时，表示反向切片，这时start应该比end的值要大才行。
"""
aList = [3, 4, 5, 6, 7, 9, 11, 13, 15, 17]
print (aList[::])  # 返回包含原列表中所有元素的新列表
print (aList[::-1])  # 返回包含原列表中所有元素的逆序列表
print (aList[::2])  # 隔一个取一个，获取偶数位置的元素
print (aList[1::2])  # 隔一个取一个，获取奇数位置的元素
print (aList[3:6])  # 指定切片的开始和结束位置
aList[0:100]  # 切片结束位置大于列表长度时，从列表尾部截断
aList[100:]  # 切片开始位置大于列表长度时，返回空列表

aList[len(aList):] = [9]  # 在列表尾部增加元素
aList[:0] = [1, 2]  # 在列表头部插入元素
aList[3:3] = [4]  # 在列表中间位置插入元素
aList[:3] = [1, 2]  # 替换列表元素，等号两边的列表长度相等
aList[3:] = [4, 5, 6]  # 等号两边的列表长度也可以不相等
aList[::2] = [0] * 3  # 隔一个修改一个
print (aList)
aList[::2] = ['a', 'b', 'c']  # 隔一个修改一个
aList[::2] = [1,2]  # 左侧切片不连续，等号两边列表长度必须相等
aList[:3] = []  # 删除列表中前3个元素

del aList[:3]  # 切片元素连续
del aList[::2]  # 切片元素不连续，隔一个删一个

```

### 4. 列表生成式、生成器表达式、字典推导式、集合推导式

```python
# 列表生成式
int_list = [1, 2, 3, 4, 5]
qu_list = [item * item for item in int_list]
print(type(qu_list))

# 生成器表达式
odd_gen = (i for i in range(5) if i % 2 == 1)
print(type(odd_gen))
odd_list = list(odd_gen)

# 字典推导式
my_dict = {"key1": "bobby1", "key2": "bobby2"}
reversed_dict = {value: key for key, value in my_dict.items()}

# 集合推导式
my_set = {key for key, value in my_dict.items()}
print(my_set)

```

### 5.序列中+ += extend 区别

1. 加号会创建一个新的序列；
2. += 符号是在原有序列的基础上修改，实际调用的是 extend 方法；
3. extend 也是在原有序列的基础上修改，实际上调用的是 append 方法。

### 6. list 与 set、dict 性能对比

|                  | Set  | List | Dict         |
| :--------------: | ---- | ---- | ------------ |
|  是否要求 Hash   | 是   | 否   | 是           |
|     查找效率     | 优   | 中   | 良           |
|       内存       | 小   | 小   | 大           |
| 是否记录元素顺序 | 否   | 是   | 是，但有变化 |

1.dict 查找的性能远远大于 list
2.在 list 中随着 list 数据的增大查找时间会增大
3.在 dict 中查找元素不会随着 dict 的增大而增大
4.dict 的 key 或者 set 的值都必须是可以 hash 的。不可变对象 都是可hash的， str， fronzenset， tuple，自己实现的类 hash
5.dict 的内存花销大，但是查询速度快， 自定义的对象或者 python 内部的对象都是用 dict 包装的
6.dict 的存储顺序和元素添加顺序有关，添加数据有可能改变已有数据的顺序

#####  fronzenset

> **frozenset()** 返回一个冻结的集合，冻结后集合不能再添加或删除任何元素。
>
> 函数签名 class frozenset([iterable])

```python
>>>a = frozenset(range(10))     # 生成一个新的不可变集合
>>> a
frozenset([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> b = frozenset('runoob') 
>>> b
frozenset(['b', 'r', 'u', 'o', 'n'])   # 创建不可变集合
>>>
```

