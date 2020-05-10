---
title: Kafka -- 集群安装与配置（Docker）
date: 2018-10-09 01:48:40
categories:
    - windows10
    - docker toolbox
    - nginx
tags:
    - docker
    - windows10
---

##### 问题描述

> 在windows10中安装docker toolbox，启动nginx，页面无法访问

##### 解决：

```bash
1.在windows上运行的docker实际是在Linux中运行
2.启动的服务中的localhost指linux中，而不是宿主机windows中的地址
3.使用docker-machine ip default 得到真正的ip是192.168.99.100
```

