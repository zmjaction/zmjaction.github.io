---
title: pyminizip -- 压缩加密文件 简单使用
date: 2020-05-09 23:08:32
categories:
    - python
    - pyminizip
tags:
    - python
    - pyminizip
---

### 使用pyminizip压缩加密文件

```python
import pyminizip
# 压缩
compression_level = 5 # 1-9
pyminizip.compress("record.db", None, "dst.zip", "password", compression_level)
# 解压缩
import pyminizip
pyminizip.uncompress("dst.zip", "password", "./", 0)
```

> 参考：
>
> <https://github.com/smihica/pyminizip>
>
> <https://stackoverflow.com/questions/17250/create-an-encrypted-zip-file-in-python/40164739>
>
> <https://stackoverflow.com/questions/59437912/how-do-i-create-a-temporary-zip-in-python-3-x-using-pyminizip>