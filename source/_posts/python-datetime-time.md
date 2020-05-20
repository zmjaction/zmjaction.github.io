---
title: python 内建模块 datetime
date: 2017-06-01 11:11:40
categories:
    - python
    - datetime
tags:
    - python
    - datetime
---

#### 获取当前时间

```python
from datetime import datetime

now = datetime.now() # 获取当前datetime
print(now) # 2020-05-20 22:07:36.825986
print(type(now))  # <class 'datetime.datetime'>
```

> `datetime.now()`返回当前日期和时间，其类型是`datetime`

<!-- more -->

#### str--> datetime 类型

```python
dt = datetime.strptime("2017-06-01 11:11:40", "%Y-%m-%d %H:%M:%S")
print(dt) # 2017-06-01 11:11:40
print(type(dt)) # <class 'datetime.datetime'>
```

#### datetime --> str

```python
from datetime import datetime
now = datetime.now()
print(now.strftime('%a, %b %d %H:%M'))  # Wed, May 20 22:23
print(now.strftime('%Y-%m-%d %H:%M:%S')) # 2020-05-20 22:23:46
```

#### datetime类型--> timestamp类型

```python
dt = datetime.strptime("2018-11-15 15:32:12", "%Y-%m-%d %H:%M:%S")
print(dt) # 2018-11-15 15:32:12
print(type(dt)) # <class 'datetime.datetime'>
print(dt.timestamp())  # 1542267132.0
```

> 注意Python的timestamp是一个浮点数。如果有小数位，小数位表示毫秒数

#### timestamp类型-->datetime类型

```python
from datetime import datetime
t = 1542267132.0
print(datetime.fromtimestamp(t))  # 本地时间
print(datetime.utcfromtimestamp(t)) # UTC时间
```

#### datetime加减

```python
from datetime import datetime,timedelta
now = datetime.now()
print(now + timedelta(hours=10))
print(now - timedelta(days=1)) 
print(now + timedelta(days=2, hours=12))
print((now - timedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')) 
```

> 使用`timedelta`你可以很容易地算出前几天和后几天的时刻

#### 时区转换

```python
from datetime import datetime, timedelta, timezone
utc_dt = datetime.utcnow().replace(tzinfo=timezone.utc) # 拿到UTC时间，并强制设置时区为UTC+0:00:
print(datetime.utcnow())
print(utc_dt)
bj_dt = utc_dt.astimezone(timezone(timedelta(hours=8)))  #  astimezone()将转换时区为北京时间:
print(bj_dt)
```

