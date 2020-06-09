---
title: postgresql -- case-- case when
mathjax: false
date: 2020-05-01 23:39:41
categories:
    - 数据库
    - postgresql
    - case
tags:
    - 数据库
    - postgresql
    - case

---

#### 1. `CASE`条件判断

```
CASE input_expression
WHEN when_expression THEN result_expression
    [ ...n ]
[ELSE else_result_expression]
END
```

`CASE`函数会判断表达式`input_expression`的值，并根据判断结果执行对应的`WHEN`分支流程

示例：

```sql
-- 创建表
create table t_test (
	id serial,
	name text,
	sex int
)
-- 插入数据
insert into t_test (name,sex) values 
('张三', '1'),
('李四', '2'),
('王五', '1'),
('小六', '2'),
('小七', '1')
-- 使用case查询
select sum(case when sex = '1' then 1 else 0 end) as 男,sum(case when sex='2' then 1 else 0 end)as 女 from t_test;
 男 | 女 
----+----
  3 |  2
(1 row)
```

#### 2. `CASE THEN`条件判断

```
CASE    
WHEN Boolean_expression THEN result_expression
    [ ...n ]
[ELSE else_result_expression]
END
```

`CASE THEN`语句与`CASE`语句类似，其条件判断语句`Boolean_expression`是一个计算结果是布尔值的表达式。

示例：

```sql
SELECT CASE WHEN 1>2 THEN 'com.itbilu' ELSE 'itbilu.com' END;
```

