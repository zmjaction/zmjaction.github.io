---
title: postman -- 上传文件
date: 2020-05-11 01:01:23
categories:
    - postman
tags:
    - postman
    - 上传文件

---

#### 1.选择post请求

```
http://127.0.0.1:9810/data/site_upload_file
```

![](http://qa7i5mbcj.bkt.clouddn.com/post.png)

<!-- more -->

#### 2.填写Headers

```
Content-Type : multipart/form-data
```

![](http://qa7i5mbcj.bkt.clouddn.com/headers.png)

#### 3.选择form-data，上传文件

![](http://qa7i5mbcj.bkt.clouddn.com/body.png)