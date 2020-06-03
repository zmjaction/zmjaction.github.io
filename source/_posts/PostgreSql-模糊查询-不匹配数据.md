---
title: postgresql -- 模糊查询
mathjax: false
date: 2020-06-03 19:39:43
categories:
    - postgresql
    - 模糊查询
tags:
    - postgresql
    - like
    - not like

---

### `like `  和 `not like`

* `like、not like`在SQL中用于模糊查询，`%`表示任意个字符，`_`表示单个任意字符
* `ESCAPE`转义使用

### `ilike` 和 `not ilike`

* `ilike`表示在模糊匹配字符串时不区分大小写，`i`即是ignore的意思。
* `not ilike`表示不模糊匹配字符串且不区分大小写。

### `~`和`~*`，`!~`和`!~*`

* `~`表示匹配正则表达式，且区分大小写。
* `~*`表示匹配正则表达式，且不区分大小写。
* `!~`是`~`的否定用法，表示不匹配正则表达式，且区分大小写。
* `!~*`是`~*`的否定用法，表示不匹配正则表达式，且不区分大小写。

### `~~`和`~~*`，`!~~`和`!~~*`

* `~~`等效于like，`~~*`等效于ilike。
* `!~~`等效于not like，`!~~*`等效于not ilike。

实例：

```sql
-- 模糊查询，不匹配的数据
SELECT
		c.id,
		c.licenseKey,
		c.siteName,
		c.topDomain,
		c.website_type,
		c.system_decide,
		c.handle_result,
		c.webmaster,
		c.webmaster_phone,
		c.webmaster_fixed_phone,
		c.organizer_unit_name,
		c.unitType,
		c.principal_person,
		c.principal_phone,
		c.principal_fixed_phone,
		c.belongingCity,
		c.organizer_unit_addr,
		to_char(c.r_time,'YYYY-MM-DD hh24:mi:ss')
FROM
		 (select * from public.t_record_detail_info where belongingcity ~ '杭州市') as c
where 1=1  and organizer_unit_addr not like  all(array['%上城区%','%下城区%','%江干区%','%拱墅区%','%滨江区%','%西湖区%','%萧山区%','%余杭区%','%富阳区%','%临安区%','%桐庐县%','%淳安县%','%建德市%'])
order by c.r_time desc
```

