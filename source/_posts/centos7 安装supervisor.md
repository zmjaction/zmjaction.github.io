---
title: centos7 安装 supervisor
date: 2017-06-01 11:11:40
categories:
    - Linux
tags:
    - Linux
    - supervisor
---

```bash
rpm -qa|grep epel-release >&/dev/null ||yum install -y epel-release
rpm -qa|grep supervisor >&/dev/null ||yum install -y supervisor
supervisord -c /etc/supervisord.conf
```

