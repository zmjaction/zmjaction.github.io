---
title: java-- 基本数据类型
mathjax: false
date: 2018-10-10 09:51:36
categories:
    - java
    - 基本数据类型
tags:
    - java
    - 基本数据类型
---

#### 基本数据类型
|  数据类型  | 大小（bit) | 范围 | 默认值 | 包装类 |
|  ----  | ----  |  ----  |  ----  |  ----  |
| byte(字节) | 8 | -128 - 127 | 0 | Byte |
| short(短整型) | 16 | -32768~32767 | 0 | Short |
| int(整型) | 32 | -2^31 ~ 2^31 -1 | 0 | Integer |
| long(长整型) | 64 | -2^63 ~ 2^63 -1 | 0L | Long |
| float(浮点型) | 32 | 1.4013E-45~3.4028E+38 | 0.0f | Float |
| double(双精度) | 64 | 4.9E-324~1.7977E+308 | 0.0d | Double |
| char(字符型) | 16 | 0-65535 / `\u0000~ \uffff` | 'u0000' | Character |
| boolean(布尔型) | 1 | true/false | false | Boolean |

#### 转换规则

范围小的类型向范围大的类型提升，`byte、short、char` 运算时直接提升为 `int `。 

```
byte、short、char‐‐>int‐‐>long‐‐>float‐‐>double
```



参考：<https://www.runoob.com/java/java-basic-datatypes.html>