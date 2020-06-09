---
title: postgresql -- 窗口函数
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

> 最近在看《深入浅出PostgreSql》想总结下窗口函数，书上示例比较少，网上搜索一番，发现了一篇好的文章，借以引用望原作者知晓，[参考连接](https://www.cnblogs.com/funnyzpc/p/9311281.html)

##### 创建表，插入原始测试数据

```sql
DROP TABLE IF EXISTS "public"."products";
CREATE TABLE "public"."products" (
    "id" varchar(10) COLLATE "default",
    "name" text COLLATE "default",
    "price" numeric,
    "uid" varchar(14) COLLATE "default",
    "type" varchar(100) COLLATE "default"
)
WITH (OIDS=FALSE);

BEGIN;
INSERT INTO "public"."products" VALUES ('0006', 'iPhone X', '9600', null, '电器');
INSERT INTO "public"."products" VALUES ('0012', '电视', '3299', '4', '电器');
INSERT INTO "public"."products" VALUES ('0004', '辣条', '5.6', '4', '零食');
INSERT INTO "public"."products" VALUES ('0007', '薯条', '7.5', '1', '零食');
INSERT INTO "public"."products" VALUES ('0009', '方便面', '3.5', '1', '零食');
INSERT INTO "public"."products" VALUES ('0005', '铅笔', '7', '4', '文具');
INSERT INTO "public"."products" VALUES ('0014', '作业本', '1', null, '文具');
INSERT INTO "public"."products" VALUES ('0001', '鞋子', '27', '2', '衣物');
INSERT INTO "public"."products" VALUES ('0002', '外套', '110.9', '3', '衣物');
INSERT INTO "public"."products" VALUES ('0013', '围巾', '93', '5', '衣物');
INSERT INTO "public"."products" VALUES ('0008', '香皂', '17.5', '2', '日用品');
INSERT INTO "public"."products" VALUES ('0010', '水杯', '27', '3', '日用品');
INSERT INTO "public"."products" VALUES ('0015', '洗发露', '36', '1', '日用品');
INSERT INTO "public"."products" VALUES ('0011', '毛巾', '15', '1', '日用品');
INSERT INTO "public"."products" VALUES ('0003', '手表', '1237.55', '5', '电器');
INSERT INTO "public"."products" VALUES ('0016', '绘图笔', '15', null, '文具');
INSERT INTO "public"."products" VALUES ('0017', '汽水', '3.5', null, '零食');
COMMIT;
```

##### **row_number()**使用

```sql
  select type,name,price,row_number() over(order by price asc) as idx from products
```

![](//tvax1.sinaimg.cn/large/d479ba17gy1gfmef0ig19j208a0cwglx.jpg)



##### Partition by

> 以上输出想要在type 类别，按照价格price 升序排序

```sql
 select type,name,price,row_number() over(PARTITION by type order by price asc) as idx from products ;
```

![](//tva4.sinaimg.cn/large/d479ba17gy1gfmei9kezrj208j0cpmxg.jpg)

##### rank()

> 以上输出，方便面和汽水价格是一样，如何将零食和汽水并列第一呢？

```sql
 SELECT type,name,price,rank() over(partition by type order by price asc) from products
```

![](//tva4.sinaimg.cn/large/d479ba17gy1gfmel68ud8j209b0cudg2.jpg)



##### **dense_rank**()

> 以上输出中，辣条排在了第三位，想要排在第二位怎么处理呢？

```sql
 SELECT type,name,price,dense_rank() over(partition by type order by price asc) from products;
```

![](//tva1.sinaimg.cn/large/d479ba17gy1gfmenzzfekj20a00cq74m.jpg)

##### **percernt_rank**()

> 限制序号在0~1之间,(0作为第一个序)

```sql
 SELECT type,name,price,percent_rank() over(partition by type order by price asc) from products
```

![](//tva1.sinaimg.cn/large/d479ba17gy1gfmeqwjuxlj20be0d30t4.jpg)

##### **cume_dist**()

> 限制序号在0~1之间相对排名

![](//tvax4.sinaimg.cn/large/d479ba17gy1gfmes99eh7j20bb0d374p.jpg)

##### **ntile(val1)**

> 限制最大序列号

```sql
SELECT type,name,price,ntile(10) over(partition by type order by price asc) from products;
```

![](//tva1.sinaimg.cn/large/d479ba17gy1gfmeud4oj3j208w0ciaae.jpg)

#####  l**ag(val1,val2,val3)**

> 获取偏移值(向下偏移)

```
SELECT id,type,name,price,lag(id,1,'') over(partition by type order by price asc) as topid from products;
```

![](//tva4.sinaimg.cn/large/d479ba17gy1gfmewwinw8j20ak0cr74p.jpg)



##### **lead(val1,val2,val3)**

> 偏移值(向上偏移) 

```sql
SELECT id,type,name,price,lead(id,1,'') over(partition by type order by price asc) as downid from products;
```

##### **first_value(val1)** 

> 第一条记录的某个字段的值

```
 SELECT id,type,name,price,first_value(name) over(partition by type order by price asc) from products;
```

![](//tva3.sinaimg.cn/large/d479ba17gy1gfmf0vqrooj20ar0cv0t7.jpg)

##### **last_value(val1)**

> 最后一条记录的某个字段

```
 SELECT id,type,name,price,last_value(name) over(partition by type order by price range between unbounded preceding and unbounded following) from products;
```

##### **nth_value(val1,val2)**

> 指定序号记录的某个字段值

```sql
SELECT id,type,name,price,nth_value(name,2) OVER(partition by type order by price range between unbounded preceding and unbounded following ) from products;
```

##### 窗口函数+聚合函数

```sql
sum(price) over (partition by type) 类别金额合计,
(sum(price) over (order by type))/sum(price) over() 类别总额占所有品类商品百分比,
round(price/(sum(price) over (partition by type rows between unbounded preceding and unbounded following)),3) 子除类别百分比,
rank() over (partition by type order by price desc) 排名,
sum(price) over() 金额总计
from products ORDER BY type,price asc;
-- 放在from后使用
select
    id,type,name,price,
    sum(price) over w1 类别金额合计,
    (sum(price) over (order by type))/sum(price) over() 类别总额占所有品类商品百分比,
    round(price/(sum(price) over w2),3) 子除类别百分比,
    rank() over w3 排名,
    sum(price) over() 金额总计
from
    products
WINDOW
    w1 as (partition by type),
    w2 as (partition by type rows between unbounded preceding and unbounded following),
    w3 as (partition by type order by price desc)
ORDER BY
    type,price asc;
```

















