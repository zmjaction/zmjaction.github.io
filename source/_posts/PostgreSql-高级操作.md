---
title: postgresql -- 高级操作
mathjax: false
date: 2020-06-04 20:23:19
categories:
    - 数据库
    - postgresql
    - 高级操作
tags:
    - 数据库
    - postgresql
    - UPSERT

---

#### 归并数据

归并数据又称为`UPSERT`语法，指的是`INSERT .. ON CONFLICT UPDATE`

> 当发生冲突的时候，可以忽略改插入操作，也可以执行指定的更新`UPDATE`操作

语法格式：

```sql
[ WITH [ RECURSIVE ] with_query [, ...] ]  
INSERT INTO table_name [ AS alias ] [ ( column_name [, ...] ) ]  
    { DEFAULT VALUES | VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }  
    [ ON CONFLICT [ conflict_target ] conflict_action ]  
    [ RETURNING * | output_expression [ [ AS ] output_name ] [, ...] ]  
```

* `ON CONFLICT DO NOTHING`:不采取任何动作
* `ON CONFLICT DO UPDATE`: 更新与要插入的行冲突的已有行
  * `conflict_target:指定冲突的目标，哪些可用约束或唯一索引产生的冲突`
  * `conflict_action:`指定一个冲突发生时的可选的动作

1.创建一张测试表:

```sql
create table test(id int primary key, name text, r_time timestamp);  
-- 注意：on conflict (必须是主键，或者唯一索引)，如果指定name 需要创建唯一索引
CREATE UNIQUE INDEX name_idx ON test (name)

```

2.不存在则插入，存在则更新

```sql
-- 插入数据
insert into test values (1,'xiaoming',now()) on conflict (id) do update set name=excluded.name,r_time=excluded.r_time; 

-- 查看数据
select * from test;

-- 通过指定id，再插入一条，发现已经被更新
insert into test values (1,'lilei',now()) on conflict (id) do update set name=excluded.name,r_time=excluded.r_time;

-- 查看数据，发现已经被更新
select * from test;
```

3.不存在则插入，存在则直接返回(不做任何处理)

```sql
-- 插入数据
insert into test values (1,'tom',now()) on conflict (id) do nothing;
insert into test values (1,'marry',now()) on conflict (id) do nothing;
insert into test values (2,'tony',now()) on conflict (id) do nothing;
-- 查看数据，发现只有tom,tony这两条数据
select * from test;
```

#### 批量插入

##### 1. `INSERT INTO ... SELECT ...`

> 注意必须是表结构相同的两个表

语法格式：

```sql
INSERT INTO test1 SELECT * from test2
```

##### 2. INSERT INTO ... VALUES(),(),()

语法格式：

```sql
INSERT INTO  
	test1 (name,r_time)
VALUES
	("tom", now()),
	("tony", now()),
	...
```

##### 3. BEGIN ... 多条insert ...; END

> 通过一个事务，插入多条sql，减少commit次数

语法格式：

```sql
BEGIN;
INSERT INTO "public"."products" VALUES ('0006', 'iPhone X', '9600', null, '电器');
INSERT INTO "public"."products" VALUES ('0012', '电视', '3299', '4', '电器');
INSERT INTO "public"."products" VALUES ('0004', '辣条', '5.6', '4', '零食');
INSERT INTO "public"."products" VALUES ('0007', '薯条', '7.5', '1', '零食');
END;
```

##### 4. COPY

语法格式：

```sql
COPY table_name [ ( column_name [, ...] ) ]
    FROM { 'filename' | PROGRAM 'command' | STDIN }
    [ [ WITH ] ( option [, ...] ) ]
```

示例：

```sql
COPY test1 from '/home/action/batch.txt'
copy tbl_test1(a,b,c) from stdin delimiter ',' csv header;

```

示例2：

```sql
-- copy 标准输出，字段间用','分隔
copy tbl_test1(a,b) to stdout delimiter ',' csv header;
```



> copy 文件--> 表
> 
>copy to 表--> 文件



#### 批量更新

语法格式：

```sql
UPDATE FROM ... VALUES(),()...
```

示例：

```sql
update test set info=tmp.info from (values (1,'new1'),(2,'new2'),(6,'new6')) as tmp (id,info) where test.id=tmp.id;
```

#### 关联更新

#### 批量删除

语法格式：

```sql
delete from test using (values (3),(4),(5)) as tmp(id) where test.id=tmp.id;  
```

#### 关联删除

#### 移动数据到历史表

#### 清空表

语法格式：

```sql
truncate test; 
```

