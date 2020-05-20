---
title: ModuleNotFoundError: No module named '_bz2'
date: 2018-11-09 10:28:31
categories:
    - python
tags:
    - python
    - bz2
---

#### ModuleNotFoundError: No module named '_bz2'

```bash
yum install bzip2-devel 
# 进入python3的安装目录,重新编译python3 cd /usr/local/Python3
./configure
make && make install
```

