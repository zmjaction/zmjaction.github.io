---
title: python -- 类和对象
date: 2019-06-10 10:20:03
categories:
    - python
tags:
    - python
    - class object
---

### 1. 类

> 和 Java 一样，所有类都默认继承 object，object 是最顶层的父类。

### 1.1 实例属性和类属性

> 实例属性通过 self 关键字定义，类属性没有 self 修饰，直接写在类中。**类属性一定要通过类名调用，不要使用对象名调用。**例子如下：

```python
class A:
    aa = 1
    def __init__(self, x, y):
        self.x = x
        self.y = y

a = A(2,3)

A.aa = 11
a.aa = 100
print(a.x, a.y, a.aa)
print(A.aa)

b = A(3,5)
print(b.aa)
```

<!-- more -->

### 1.2 私有属性和私有方法

> Python 中没有 private 关键字，属性或方法的私有都是通过双下划线完成的。定义为私有的属性和方法不能被子类继承。
> 题外话，Python 中会将双下划线修饰的属性变形为 _类名__属性名 这种命名方式，如果主动使用这种方式也可以调用到想要的属性，但是开发中不要这样做，调用私有属性需要通过提供的 getter 和 setter 方法

```python
from class_method import Date
class User:
    def __init__(self, birthday):
        self.__birthday = birthday

    def get_age(self):
        #返回年龄
        return 2018 - self.__birthday.year


if __name__ == "__main__":
    user = User(Date(1990,2,1))
    print(user._User__birthday)
    print(user.get_age())
```

### 1.3 构造函数

> 构造函数通过 **init** 方法定义。例子如下：

```python
class Date:
    # 构造函数
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
```

## 1.4 获取类或者对象的内部结构

1. 方法一：**dict** 能够获取类或者对象的内部结构。
2. 方法二：使用 dir 内置函数，这种方式更加强大。

> 例子如下：

```python
class Date:
    # 构造函数
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
class User:
    def __init__(self, birthday):
        self.__birthday = birthday

    def get_age(self):
        #返回年龄
        return 2018 - self.__birthday.year


if __name__ == "__main__":
    print(Date(1990,2,1))
    user = User(Date(1990,2,1))
    print(user._User__birthday)
    print(user.get_age())
```

### 1.5 判断一个对象是否具有某个属性

> hasattr 方法

```python
#我们去检查某个类是否有某种方法
class Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list

    def __len__(self):
        return len(self.employee)


com = Company(["bobby1","bobby2"])
print(hasattr(com, "__len__"))
```

### 静态方法、类方法、对象方法以及参数

- 静态方法：
  - 格式：在方法上面添加 @staticmethod
  - 参数：静态方法可以有参数也可以无参数
  - 应用场景：一般用于和类对象以及实例对象无关的代码。
  - 使用方式： 类名.类方法名(或者对象名.类方法名)。
- 类方法：
  - 在方法上面添加@classmethod
  - 方法的参数为 cls 也可以是其他名称，但是一般默认为cls
  - cls 指向 类对象
  - 应用场景：当一个方法中只涉及到静态属性的时候可以使用类方法(类方法用来修改类属性)。
  - 使用可以是 对象名.类方法名。或者是 类名.类方法名
- 实例方法：又叫对象方法，指类中定义的普通方法

```python
class Date:
    # 构造函数
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    def tomorrow(self):
        self.day += 1

    @staticmethod
    def parse_from_string(date_str):
        year, month, day = tuple(date_str.split("-"))
        return Date(int(year), int(month), int(day))

    @staticmethod
    def valid_str(date_str):
        year, month, day = tuple(date_str.split("-"))
        if int(year) > 0 and (int(month) > 0 and int(month) <= 12) and (int(day) > 0 and int(day) <= 31):
            return True
        else:
            return False

    @classmethod
    def from_string(cls, date_str):
        year, month, day = tuple(date_str.split("-"))
        return cls(int(year), int(month), int(day))

    def __str__(self):
        return "{year}/{month}/{day}".format(year=self.year, month=self.month, day=self.day)


if __name__ == "__main__":
    new_day = Date(2018, 12, 31)
    new_day.tomorrow()
    print(new_day)

    # 2018-12-31
    date_str = "2018-12-31"
    year, month, day = tuple(date_str.split("-"))
    new_day = Date(int(year), int(month), int(day))
    print(new_day)

    # 用staticmethod完成初始化
    new_day = Date.parse_from_string(date_str)
    print(new_day)

    # 用classmethod完成初始化
    new_day = Date.from_string(date_str)
    print(new_day)

    print(Date.valid_str("2018-12-32"))
```



### 2. 继承 inheritance

### 2.1 继承的实现

> Python 的继承不需要关键字，只需要在类名后面直接添加父类名称即可。例子如下：

```python
class A:
    def __init__(self):
        print ("A")

class B(A):
    def __init__(self):
        print ("B")
        super().__init__()
```

### 2.2 判断对象是否是一个类的实例

> Python 中提供了 isinstance 关键字完成这一需求，类似于 Java 中 instanceof 关键字。不要使用 type 配合 is 关键字的方式。 isintance 会查找继承链，而 is 不会，is 只判断对象是否指向同一个内存地址。

```python
class A:
    pass

class B(A):
    pass

b = B()

print(isinstance(b, B))
print(isinstance(b, A))

print(type(b) is A)
```

### 2.3 super 真的是调用父类吗？（重点）

> 答案：不是，super 调用的是 **mro** 函数的下一个类的构造函数。MRO 全称 Method Resolution Order。

```python
class B(A):
    def __init__(self):
        print("B")
        super().__init__()


class C(A):
    def __init__(self):
        print("C")
        super().__init__()


class D(B, C):
    def __init__(self):
        print("D")
        super(D, self).__init__()


if __name__ == "__main__":
    print(D.__mro__)
    d = D()
```

>由于 Python 允许多继承，多继承会存在菱形继承问题，会出现如下两种类型：
>类型一：

![](https://img-blog.csdnimg.cn/20190508113158612.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

>上图中，A 同时继承 B 和 C，B 和 C 又都继承 D，假设 C 覆盖了 D 中的某个方法，记该方法为 M。在 Python 2 的非常早期版本中使用的是 DFS 方法搜索类的方法继承关系，使用 DFS 方法将导致 A 类的对象始终无法调用 C 类中重写的方法 M，调用的始终是 D 类中原始的方法 M。为了解决这个问题，Python 后来将 DFS 算法更新为 BFS 算法，广度优先搜索算法虽然能解决类型一的继承问题，但是却解决不了类型二这种情形。
>类型二：

![](https://img-blog.csdnimg.cn/20190508113648949.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215b25lbG90dXM=,size_16,color_FFFFFF,t_70)

>上图中，A 同时继承类 B 和 C，B 和 C 又分别继承 D 和 E 类，假设 D 类和 C 类中有同名方法 M。如果使用 BFS 方法会导致 A 类的对象依然会调用 C 类中的 M 方法，而这就导致了错误调用。正确的调用应该是 D 类中的 M 方法，因为 A 首先继承 B 类，B 类中存在它从 D 类中继承过来的方法 M，应该返回的调用方法是 B 类从 D 类继承而来的方法，而不是 C 类中的方法 M。
>Python 现在的算法采用的是 C3 算法，解决了上述的两种菱形继承问题。

### 2.4 mixin 模式

django rest framework中对多继承使用的经验

> 尽管 Python 支持多继承，但是通常都不建议使用多继承方式。如果有多继承需求，通常采用 mixin 模式解决。
> mixin模式特点:

1. Mixin类功能单一
2. 不和基类关联，可以和任意基类组合，基类可以不和mixin关联就能初始化成功
3. 在mixin中不要使用super这种用法
4. 使用__bases__将需要继承的类组合在一起

```python
def mixin(pyClass, pyMixinClass, key=0):
    if key:
        pyClass.__bases__ = (pyMixinClass,) + pyClass.__bases__
    elif pyMixinClass not in pyClass.__bases__:
        pyClass.__bases__ += (pyMixinClass,)
    else:
        pass


class test1:
    def test(self):
        print('In the test1 class!')


class testMixin:
    def test(self):
        print('In the testMixin class!')


class test2(test1, testMixin):
    def test(self):
        print('In the test2 class!')


class test0(test1):
    pass


if __name__ == '__main__':
    print(test0.__mro__)  # 继承了test1，object
    test_0 = test0()
    test_0.test()  # 调用test1的方法
    mixin(test0, testMixin, 1)  # 优先继承testMixin类
    test__0 = test0()
    test__0.test()  # 由于优先继承了testMixin类，所以调用testMixin类的方法
    print(test0.__mro__)

    print(test2.__mro__)
    mixin(test2, testMixin)     # test2中已经继承了testMixin
    print(test2.__mro__)
```

### 11.python中的with语句

普通的try...finally语句的调用流程：

- 如果无异常：try->else->finally
  - finally中有return，则调用finally中的return
  - finally中无return
    - try中有return，则调用try中return
    - try中无return，则调用else中的return
- 如果有异常：try->except->finally
  - finally中有return，则调用finally中的return
  - finally中无return，则调用except中的return

```python
# try except finally
def exe_try():
    try:
        print("code started")
        # raise KeyError
        return 1
    except KeyError as e:
        print("key error")
        return 2
    else:
        print("other error")
        return 3
    finally:
        print("finally")
        return 4


if __name__ == "__main__":
    result = exe_try()   
    print(result)　　# 4
```

上下文管理器协议：with

```python
上下文管理器协议：with
两个魔法函数：__enter__、__exit__
    __enter__：进入时执行，用于获取资源
    __exit__：结束时执行，用于释放资源
# 上下文管理器协议
class Sample:
    def __enter__(self):
        print("enter")
        # 获取资源
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        # 释放资源
        print("exit")

    @staticmethod
    def do_something():
        print("doing something")


with Sample() as sample:
    sample.do_something()   # enter, doing something, exit
```

### contextlib简化上下文管理器

```python
1.将__enter__、__exit__以及中间处理函数都整合到一个函数中
2.yield：中间处理函数
3.使用装饰器：@contextlib.contextmanager
```

```python
import contextlib

@contextlib.contextmanager
def file_open(file_name):
    print("file open")
    yield {}
    print("file end")

with file_open("bobby.txt") as f_opened:
    print("file processing")  # file open, file processing, file end
```





   

### 3. 多态 polymorphism

### 3.1 多态

> 在 Java 语言中，多态的实现是类之间必须继承关系，将子类实例化的对象赋值给父类，从而实现多态。而在 Python 语言中，不需要继承关系，**类之间只需要有共同的方法**即可，也就是所谓的鸭子类型。注意仔细体会下面的例子：

```python
class Cat(object):
    def say(self):
        print("i am a cat")

class Dog(object):
    def say(self):
        print("i am a fish")

class Duck(object):
    def say(self):
        print("i am a duck")

animal_list = [Cat, Dog, Duck]
for animal in animal_list:
    animal().say()
```

### 3.2 鸭子类型（在定义一个类时 有共同的方法）

> 理解了上面的例子，我们来进一步理解鸭子类型。鸭子类型贯穿了 Python 语言的始终，关系到整个语言面向对象的设计理念。当我们定义了一个类之后，只要实现了魔法函数（以双下划线开始和结束的方法）， Python 就可以识别出来，进而产生多态。下面的 Company 类实现了 getitem 和 len，故而可以被当做是一个 iteratable 类型。
>

```python
class Cat(object):
    def say(self):
        print("cat")


class Dog(object):
    def say(self):
        print("dog")


class Duck(object):
    def say(self):
        print("duck")


animal_list = [Cat, Dog, Duck]
for animal in animal_list:
    animal().say()      # cat dog duck，具有多态性
a = ["bobby1", "bobby2"]

b = ["bobby2", "bobby"]
name_tuple = ["bobby3", "bobby4"]
name_set = set()
name_set.add("bobby5")
name_set.add("bobby6")
a.extend(name_set) # 由于extend 中可以接收iterable类型，所以传递name_tuple，name_set，b都是可以的，这些都是iterable类型
print(a)
---------------------------------------------------------------

class Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list

    def __getitem__(self, item):
        return self.employee[item]

    def __len__(self):
        return len(self.employee)

company = Company(["tom", "bob", "jane"])

dog = Dog()
a = ["bobby1", "bobby2"]

b = ["bobby2", "bobby"]
name_tuple = ["bobby3", "bobby4"]
name_set = set()
name_set.add("bobby5")
name_set.add("bobby6")
a.extend(company) # 。。。。。
print(a)

```

### 4. 抽象基类

> 通过前面的学习，我们知道 Python 语言的多态都是围绕鸭子类型设计的，并不需要继承某一父类，只需要实现特定的方法即可，那么为什么还会有抽象基类这一概念呢？这一问题的答案，我们通过两个需求来说明。

1. 判断一个对象是否是一个类的实例
2. 强制某个子类必须实现某些方法

```python
#我们去检查某个类是否有某种方法
class Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list

    def __len__(self):
        return len(self.employee)


com = Company(["bobby1","bobby2"])
print(hasattr(com, "__len__"))

#我们在某些情况之下希望判定某个对象的类型
from collections.abc import Sized
isinstance(com, Sized)
```

> 当子类调用 set 方法时，就会报错。**但是，这种方式的缺点是必须调用时才会报错。**如果想实例化时，没有实现特定的方法就报错呢？此时就需要使用 Python 的 abc 模块。

```python
import abc
from collections.abc import *

class CacheBase(metaclass=abc.ABCMeta):
    @abc.abstractmethod
    def get(self, key):
        pass

    @abc.abstractmethod
    def set(self, key, value):
        pass

class RedisCache(CacheBase):
    def get(self, key):
        print("get")

    def set(self, key, value):
        print("set")

redis_cache = RedisCache()
redis_cache.set("name", "wzh")  # set
```

