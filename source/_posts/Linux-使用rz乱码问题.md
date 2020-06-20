---
title: 解决使用 rz 命令乱码问题
date: 2018-05-20 10:20:40
categories:
    - Linux
    - rz
tags:
    - Linux
    - rz
---

#### 解决使用 rz 命令乱码问题

###### **使用 rz 命令上传一个 tar 包出现一堆乱七八糟的乱码，这个时候使用**

```bash
rz -be
或者
rz -e
解析：-e
Force sender to escape all control characters; normally XON, XOFF, DLE, CR-@-CR, and Ctrl-X are escaped.
```

