---
title: python -- 魔法函数
date: 2019-05-10 09:08:23
categories:
    - python
tags:
    - python
    - 魔法函数
---

#### 1.什么是魔法函数

- 魔法函数是以上下划线开头和双下划线结尾的函数：__init__(self){...}
- 魔法函数是python帮我们定义过的不需要自己定义
- python可以帮我们自动（隐式）调用魔法函数，不需要自己调用：例如调用__getitem__(self,item)就具有了迭代器属性可以放入for in 循环中遍历

```python
class Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list

    # python编译器自动调用，使得可以在for in循环中使用
    def __getitem__(self, item):
        return self.employee[item]


company = Company(["tom", "bob", "jane"])
for em in company:
    print(em)       # tom bob jane
```

<!-- more -->

#### 2.python的数据模型以及数据模型对python的影响

- 此处的数据模型也就是常说的魔法函数
- 如果自己没有实现类中的一些数据模型可能会对python中的遍历，切片等语法产生影响

```python
class Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list

    # python编译器自动调用，使得可以在for in循环中使用
    def __getitem__(self, item):
        return self.employee[item]

    # python编译器自动调用，获得长度
    # def __len__(self):
    #     return len(self.employee)


company = Company(["tom", "bob", "jane"])
for em in company:
    print(em)       # tom bob jane

# __getitem__处理切片的数据可以得到len，不可以处理没有切片的数据
company1 = company[:2]

# 步骤：
# 1.查找否有__len__
# 2.没有__len__再查看数据是否切片并且是否有__getitem__
# 3.有则可以获得len，如果数据没有切片则会报错
print(len(company1))    # 2
```

#### 3.魔法函数一览表

```python
非数学运算
字符串：
__repr__
__str__
集合、序列相关
__len__
__getitem__
__setitem__
__delitem__
__contains__
迭代相关
__iter__
__next__
可调用
__call__
with上下文管理器
__enter__
__exit__
数值转换
__abs__
__bool__
__int__
__float__
__hash__
__index__
元类相关
__new__
__init__
属性相关
__getattr__、__setattr__
__getattribute__、setattribute__
__dir__
属性描述符
__get__
__set__
__delete__
协程
__await__
__aiter__
__anext__
__aenter__
__aexit__
数学运算
一元运算符
__neg__(-)、__pos__(+)、__abs__
二元运算符
__lt__(<)、__le__(<=)、__eq__(==)、__ne__(!=)、__gt__(>)、__ge__(>=)
__add__(+)、__sub__(-)、__mul__(*)、__truediv__(/)、__floordiv__(//)、
算术运算符
反向算术运算符
增量赋值算数运算符
位运算符
反向位运算符
增量赋值位运算符
```

#### 编写__str__、__repr__方法

```python
class Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list

    # 在对象格式化时调用
    def __str__(self):
        return ",".join(self.employee)

    # 在开发模式下调用
    def __repr__(self):
        return ",".join(self.employee)


company = Company(["tom", "bob", "jane"])
# 对象格式化
print(company)  # tom,bob,jane
# 开发模式:cmd或notebook中调用
company     # tom,bob,jane
```

#### 编写__add__方法

```python
class MyVector(object):
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        re_vector = MyVector(self.x + other.x, self.y + other.y)
        return re_vector

    def __str__(self):
        return "x:{x},y:{y}".format(x=self.x, y=self.y)


vec1 = MyVector(1, 2)
vec2 = MyVector(3, 4)
print(vec1 + vec2)  # x:4,y:6
```





   

