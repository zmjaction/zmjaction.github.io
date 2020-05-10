---
title: python -- 对象引用-可变性-垃圾回收
date: 2019-06-20 02:10:23
categories:
    - python
tags:
    - python
    - gc
---

### python中的数据类型

### （1）python3中有六个标准的数据类型

- Number（数值）：int、float、bool、complex

- String（字符串）： str = 'Runoob' 

- List（列表）： list = [ 'abcd', 786 , 2.23, 'runoob', 70.2 ] 

- Tuple（元组）： tuple = ( 'abcd', 786 , 2.23, 'runoob', 70.2 ) 

- Set（集合）：

  ```
  {'Mary', 'Jim', 'Rose', 'Jack', 'Tom'}
  ```

- Dictionary（字典）：

  ```
  tinydict = {'name': 'runoob','code':1, 'site': 'www.runoob.com'}
  ```

<!-- more -->

### python中的可变类型和不可变类型：

| 不可变类型       | 可变类型     |
| ---------------- | ------------ |
| Number（数字）   | List（列表） |
| String（字符串） | Dic（字典）  |
| Tuple（元组）    | Sets（集合） |

 注：python的参数传递：

- 不可变对象：传值
- 可变对象：传引用

# 1.python变量是什么

- python的变量**实质是一个指针**，而java普通变量是一个容器直接存入值。

为什么b也变了呢，由于a，b同时指向同一个地址，导致a指向的内容改变也会让b改变，id()获得对象所指向的内存中的地址，如果是对象本身的地址的话a,b应该是不相同的。

```
a = [1, 2, 3]
b = a
# a和b实质是指针都指向同一个地址，修改a同时会修改b
print(id(a), id(b))  # 54866344 54866344
print(a is b)   # True

a.append(2)
print(a, b)  # [1, 2, 3, 2] [1, 2, 3, 2]
```

### 注：python变量生成步骤

- 先建立对象[1,2,3,4] 赋值一个地址：19333544
- 让a指向这个19333544地址
- 再建立对象[1,2,3,4] 赋值一个地址：193335480
- 让b指向这个193335480地址
- 让c指向与a指向相同的地址

a和b指向不同的对象，c和a指向相同对象

a和b指向不同的对象，c和a指向相同对象

```
1 a = [1, 2, 3, 4]
2 b = [1, 2, 3, 4]
3 c = a
4 print(id(a), id(b), id(c))  # 19333544 19333480 19333544
```

**关键点：**

**对于不可变类型都指向同一个地址，可变类型都指向不同地址**

```
a = 1
b = 1
c = a
print(id(a), id(b), id(c))  # 1399769008 1399769008 1399769008

a = "abc"
b = "abc"
c = a
print(id(a), id(b), id(c))  # 24048032 24048032 24048032
```

---------------------------------------------------------------------------------------------------

* 1.== 和 is 的区别
* 2.垃圾回收
* 3.一个经典的参数引用错误

### 1. == 和 is 的区别

> == 判断符是调用类的 **eq** 方法，is 是调用 id() 判断 id 是否相等。

```python
a = [1,2,3,4]
b = [1,2,3,4]

class People:
    pass

person = People()
# 也可以用 isinstance
if type(person) is People:
    print ("yes")
# True
print(a == b)
# id 不同
print (id(a), id(b))
# False
print (a is b)

```

### 2. 垃圾回收

> 在 C++ 语言中，delete 关键字直接将一个变量回收了，但是在 Python 中，**del 关键字是将一个变量的引用计数器减一，当减到0的时候，就会回收该变量。**对于自定义类，del 关键字是调用类中的 **del** 方法。
>
> - CPython使用垃圾回收算法是引用计数的方式
>
> 先建立a，让b，c同时指向a所指向的内存此时引用计数值为3，del a之后引用计数为2，最后程序结束删除对象

```python
class A:
    def __init__(self, name):
        self.name = name
        print("init:" + name)

    def __del__(self):
        print("del:" + self.name)


a = A("a")  # init:a
b = a
c = a
del a
c.name = "c"
print(b.name)  # c
b.name = "b"
print(b.name)  # b  
最后执行del:b
```



### 3. 一个经典的参数引用错误

> 下面的例子诠释了传递 list 导致的对象成员值相互污染。解决办法是：传递 tuple。

```python
class Company:
    def __init__(self, name, staffs=[]):
        self.name = name
        self.staffs = staffs
    def add(self, staff_name):
        self.staffs.append(staff_name)
    def remove(self, staff_name):
        self.staffs.remove(staff_name)

if __name__ == "__main__":
    com1 = Company("com1", ["bobby1", "bobby2"])
    com1.add("bobby3")
    com1.remove("bobby1")
    print (com1.staffs)

    com2 = Company("com2")
    com2.add("bobby")
    print(com2.staffs)

    print (Company.__init__.__defaults__)

    com3 = Company("com3")
    com3.add("bobby5")
	
	# 下面会发现 com2 和 com3 的 staffs 成员值一样，这是由于 list 是可变对象的缘故
    print (com2.staffs)
    print (com3.staffs)
    print (com2.staffs is com3.staffs)

```