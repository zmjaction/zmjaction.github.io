---
title: python -- 元类编程
date: 2019-06-25 23:23:23
categories:
    - python
tags:
    - python
    - property
    - getattr
    - getattribute
    - mateclass

---

### 1. property 动态属性

> 使用 property 注解，可以将某个方法装饰属性描述符，将取函数变成取属性的方式。那么，什么时候会用到呢？ 以下面的代码为例，假设项目早期的版本代码中有 age 这个属性，随着项目的迭代，age 这个属性后期被移除了，添加了新的计算逻辑，那么之前的代码中通过 self.age 这种方式都需要被修改。此时，通过 property 注解，就可以将新写的 age 函数变成属性，之前的代码旧代码旧不用修改了。
> 通过 property 可以向属性一样获取值，那么如何设置值呢？答案是使用 setter 装饰器。

<!-- more -->

```python
from datetime import date, datetime
class User:
    def __init__(self, name, birthday):
        self.name = name
        self.birthday = birthday
        # self.age = 0
        self._age = 0

    @property
    def age(self):
        return datetime.now().year - self.birthday.year

    @age.setter
    def age(self, value):
        self._age = value

if __name__ == "__main__":
    user = User("bobby", date(year=1987, month=1, day=1))
    # 通过 setter 方式设置值
    user.age = 30
    print (user._age)
    # 通过 property 方式获取 age 函数返回的值
    print(user.age)
-----------------------------------------------------------------------------------
from werkzeug.security import check_password_hash, generate_password_hash
class ProcessPwd:
    def __init__(self):
        self.password_hash = ""

    @property
    def password(self):
        raise AttributeError("当前属性不允许读取")

    @password.setter
    def password(self, value):
        self.password_hash = generate_password_hash(value)

    def check_password(self, pwd_hash, password):
        """校验密码"""
        return check_password_hash(pwd_hash, password)
if __name__ == '__main__':
    pwd = ProcessPwd()
    pwd.password = "123456"
    print(pwd.password_hash)
    print(check_password_hash(pwd.password_hash, "123456"))
```

### 2. **getattr** 和 **getattribute**

> 当调用某个不存在的属性是，Python 会调用 **getattr** 函数。
> 当调用任何一个属性时，Python 都会调用 **getattribute** 函数。**通常不建议重写该方法**，该方法把持了所有调用属性的入口，如果重写的不好，可能会弄巧成拙。

```python
from datetime import date
class User:
    def __init__(self, name, birthday):
        # self.name = name
        self.birthday = birthday

    # 当查找不到这个属性的时候会调用__getattr__
    def __getattr__(self, item):
        return "not attr"

    # 所有属性的入口
    # def __getattribute__(self, item):
    #     return "bobby"


if __name__ == '__main__':
    user = User("he", date(year=1999, month=1, day=1))
    print(user.name)
---------------------------------------------------------------------------------------
from datetime import date
class User:
    def __init__(self,info={}):
        self.info = info

    def __getattr__(self, item):
        return self.info[item]

    def __getattribute__(self, item):
        return "bobby"

if __name__ == "__main__":
    user = User(info={"company_name":"imooc", "name":"bobby"})
    print(user.test)
```

### 3. 属性描述符

> 一个项目中往往包含几十张表，每个表里面都会有非常多的字段，所以类中就会包含大量 property 和 setter 这种代码，导致大量重复代码，那么，怎么解决复用问题呢？这时就需要属性描述符了。属性描述符类需要实现：**get**, **set**, **delete** 三个方法。

```python
from datetime import date, datetime
import numbers

class IntField:
    #数据描述符
    def __get__(self, instance, owner):
        return self.value
    def __set__(self, instance, value):
        if not isinstance(value, numbers.Integral):
            raise ValueError("int value need")
        if value < 0:
            raise ValueError("positive value need")
        self.value = value
    def __delete__(self, instance):
        pass


class NonDataIntField:
    #非数据属性描述符
    def __get__(self, instance, owner):
        return self.value

class User:
    age = IntField()
    # age = NonDataIntField()

'''
如果user是某个类的实例，那么user.age（以及等价的getattr(user,’age’)）
首先调用__getattribute__。如果类定义了__getattr__方法，
那么在__getattribute__抛出 AttributeError 的时候就会调用到__getattr__，
而对于描述符(__get__）的调用，则是发生在__getattribute__内部的。
user = User(), 那么user.age 顺序如下：

（1）如果“age”是出现在User或其基类的__dict__中， 且age是data descriptor， 那么调用其__get__方法, 否则

（2）如果“age”出现在user的__dict__中， 那么直接返回 obj.__dict__[‘age’]， 否则

（3）如果“age”出现在User或其基类的__dict__中

（3.1）如果age是non-data descriptor，那么调用其__get__方法， 否则

（3.2）返回 __dict__[‘age’]

（4）如果User有__getattr__方法，调用__getattr__方法，否则

（5）抛出AttributeError

'''

# class User:
#
#     def __init__(self, name, email, birthday):
#         self.name = name
#         self.email = email
#         self.birthday = birthday
#         self._age = 0
#
#     # def get_age(self):
#     #     return datetime.now().year - self.birthday.year
#
#     @property
#     def age(self):
#         return datetime.now().year - self.birthday.year
#
#     @age.setter
#     def age(self, value):
#         #检查是否是字符串类型
#         self._age = value

if __name__ == "__main__":
    user = User()
    user.__dict__["age"] = "abc"
    print (user.__dict__)
    print (user.age)
    # print (getattr(user, 'age'))
    # user = User("bobby", date(year=1987, month=1, day=1))
    # user.age = 30
    # print (user._age)
    # print(user.age)
```

### 4. __new__和__init__的区别

1. new 是用来控制对象的生成过程， 在对象生成之前。
   1. new 为对象分配内存空间，返回对象的引用
   2. init 对象初始化，定义实例属性
2. init 是用来完善对象的。
3. 如果 new 方法不返回对象， 则不会调用 init 函数。

```python
class User:
	# 第一个参数使 cls
    def __new__(cls, *args, **kwargs):
        print (" in new ")
        return super().__new__(cls)
    # 第一个参数使 self
    def __init__(self, name):
        print (" in init")
        pass
a = int()
#new 是用来控制对象的生成过程， 在对象生成之前
#init是用来完善对象的
#如果new方法不返回对象， 则不会调用init函数
if __name__ == "__main__":
    user = User(name="bobby")
-----------------------------------------------------------------------------------------
# 单例模式
class MusicPlayer(object):

    # 记录第一个被创建对象的引用
    instance = None
    # 记录是否执行过初始化动作
    init_flag = False
    def __new__(cls, *args, **kwargs):
        # 1. 判断类属性是否是空对象
        if cls.instance is None:
            # 2. 调用父类的方法，为第一个对象分配空间
            cls.instance = super().__new__(cls)
        # 3. 返回类属性保存的对象引用
        return cls.instance

    def __init__(self):

        if not MusicPlayer.init_flag:
            print("初始化音乐播放器")

            MusicPlayer.init_flag = True

# 创建多个对象
player1 = MusicPlayer()
print(player1)

player2 = MusicPlayer()
print(player2)
```

### 5. 动态创建类

### 5.1 使用工厂模式创建类

> 通过手工预先编写预置类的方式创建动态类

```python
def create_class(name):
    if name == "user":
        class User:
            def __str__(self):
                return "user"
        return User
    elif name == "company":
        class Company:
            def __str__(self):
                return "company"
        return Company

if __name__ == "__main__":
    # 通过 user 参数，创建了 User 类
    MyClass = create_class("user")
    my_obj = MyClass()
    print(type(my_obj))
```

### 5.2 使用 type 动态创建类

```python
#type动态创建类

# 动态方法必须接受 self 参数
def say(self):
    return "i am user"
    # return self.name

class BaseClass():
    def answer(self):
        return "i am baseclass"

if __name__ == "__main__":
	# 第一个参数：类的名称；
	# 第二个参数：基类名；
	# 第三个参数：属性名及其对应值，或者方法名及其对应的方法逻辑
	# 注意下面一行中，第二个 say 不要写成 say(), 尽管这是一个方法，也不要写成 say()
    User = type("User", (BaseClass, ), {"name": "user", "say": say})
    my_obj = User()
    print(type(my_obj))
    print(my_obj.name)
```

### 5.3 元类方式动态创建类

> 首先，我们需要弄明白什么是元类。元类是创建类的类，也就是 type。type 创建了 class，class 创建了对象。即，类也是一种类。懂 Java 反射机制的同学非常容易理解这些内容。

### 5.3.1 只有 metaclass 时

> 当只有 metaclass 时，创建过程：type --> class(对象) --> 对象
> 

```python
class MetaClass(type):
	# 在 metaclass 时，需要将 *args, **kwargs 向上传递，否则会报错
    def __new__(cls, *args, **kwargs):
        return super().__new__(cls, *args, **kwargs)

# 什么是元类， 元类是创建类的类。
# 创建过程：type --> class(对象) --> 对象 
class User(metaclass=MetaClass):
    def __init__(self, name):
        self.name = name
    def __str__(self):
        return "user"
# python中类的实例化过程，会首先寻找metaclass，通过metaclass去创建user类
# type 去创建类对象，实例
```

### 5.3.2 当既有基类又有 metaclass 时

> 当既有基类又有 metaclass 时，会首先找 metaclass，现在当前的类中找 metaclass，如果找不带， 就在 BaseClass 中找 metaclass，如果还找不到，就在模块中找 metaclass。如果都找不到就调用 type 创建。

```python
class BaseClass():
    def answer(self):
        return "i am baseclass"

class MetaClass(type):
    def __new__(cls, *args, **kwargs):
        return super().__new__(cls, *args, **kwargs)

# 什么是元类， 元类是创建类的类。
# 创建过程：type --> class(对象) --> 对象 
class User(BaseClass, metaclass=MetaClass):
    def __init__(self, name):
        self.name = name
    def __str__(self):
        return "user"
```

## 6. 装饰器

```python
def debug(func):
   def wrapper():
       print "[DEBUG]: enter {}()".format(func.__name__)        
       return func()    
   return wrapper

@debug
def say_hello():
   print "hello!"
————————————————

def debug(func):
   def wrapper(*args, **kwargs):  # 指定宇宙无敌参数
       print "[DEBUG]: enter {}()".format(func.__name__)        
       print 'Prepare and say...',        
       return func(*args, **kwargs)    
   return wrapper  # 返回

@debug
def say(something):
   print "hello {}!".format(something)
————————————————

def logging(level):
   def wrapper(func):
       def inner_wrapper(*args, **kwargs):
           print "[{level}]: enter function {func}()".format(
               level=level,
               func=func.__name__)            
           return func(*args, **kwargs)        
       return inner_wrapper    
   return wrapper
   
@logging(level='INFO')
def say(something):
   print "say {}!".format(something)

@logging(level='DEBUG')
def do(something):
   print "do {}...".format(something)

if __name__ == '__main__':
   say('hello')
   do("my work")
————————————————

class logging(object):
   def __init__(self, func):
       self.func = func    
   def __call__(self, *args, **kwargs):
       print "[DEBUG]: enter function {func}()".format(
           func=self.func.__name__)        
       return self.func(*args, **kwargs)

@logging
def say(something):
   print "say {}!".format(something)
————————————————

class logging(object):
   def __init__(self, level='INFO'):
       self.level = level    
   def __call__(self, func):  # 接受函数
       def wrapper(*args, **kwargs):
           print "[{level}]: enter function {func}()".format(
               level=self.level,
               func=func.__name__)
           func(*args, **kwargs)        
       return wrapper  # 返回函数

@logging(level='INFO')
def say(something):
   print "say {}!".format(something)
————————————————

```

