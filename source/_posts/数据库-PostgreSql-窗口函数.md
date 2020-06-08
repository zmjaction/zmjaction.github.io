---
title: postgresql -- 模糊查询
mathjax: false
date: 2020-06-03 19:39:43
categories:
	- 数据库
    - postgresql
    - 窗口函数
tags:
	- 数据库
    - postgresql
    - 窗口函数

---

#### 窗口函数

* 1. 与使用聚合函数执行的计算类似
* 2. 不会使多行聚合成一行，与聚合的函数的区别

##### 窗口函数的语法

```sql
function_name ([expression [, expression ... ]]) OVER ( window_definition )
window_definition语法：
	[PARTITION BY expression [,...]]
	[ORDER BY expression [ASC|DESC|USING operator] [NULLS {FIRST|LAST}] [,...] ]
```

* 1.  总是包含一个`OVER`子句，即窗口函数的名称和参数, 决定究竟将 查询的行如何通过窗口函数拆分处理

* 2. `partition by` ：对结果集进行分组

* 3. `OVER`不使用`PARTITION BY`时即代表整个表

* 4. `order by`：设定结果集的分组数据排序




| 函数                                                        | 返回类型           | 描述                                                         |
| ----------------------------------------------------------- | ------------------ | ------------------------------------------------------------ |
| `row_number()`                                              | `bigint`           | 在其分区中的当前行号，从1计                                  |
| `rank()`                                                    | `bigint`           | 有间隔的当前行排名；与它的第一个相同行的`row_number`相同     |
| `dense_rank()`                                              | `bigint`           | 没有间隔的当前行排名；这个函数计数对等组。                   |
| `percent_rank()`                                            | `double precision` | 当前行的相对排名: (`rank` - 1) / (总行数 - 1)                |
| `cume_dist()`                                               | `double precision` | 当前行的相对排名：(前面的行数或与当前行相同的行数)/(总行数)  |
| `ntile(*num_buckets* integer)`                              | `integer`          | 从1到参数值的整数范围，尽可能相等的划分分区。                |
| `lag(*value* any [, *offset* integer [, *default* any ]])`  | `类型同 value`     | 计算分区当前行的前`*offset*` 行，返回`*value*` 。如果没有这样的行， 返回`*default*`替代。 `*offset*`和`*default*` 都是当前行计算的结果。如果忽略了，则`*offset*` 默认是1，`*default*`默认是 null。 |
| `lead(*value* any [, *offset* integer [, *default* any ]])` | `类型同value`      | 计算分区当前行的后`*offset*`行， 返回`*value*`。如果没有这样的行， 返回`*default*`替代。 `*offset*`和`*default*` 都是当前行计算的结果。如果忽略了，则`*offset*` 默认是1，`*default*`默认是 null。 |
| `first_value(*value* any)`                                  | `类型同value`      | 返回窗口第一行的计算`*value*`值。                            |
| `last_value(*value* any)`                                   | `类型同value`      | 返回窗口最后一行的计算`*value*`值。                          |
| `nth_value(*value* any, *nth* integer)`                     | `类型同value`      | 返回窗口第`*nth*`行的计算 `*value*`值（行从1计数）；没有这样的行则返回 null。 |

