---
title: hexo themes/next主题文件无法提交github
date: 2016-07-09 01:48:40
categories:
    - hexo
tags:
    - hexo
    - themes/next
    - github
---

#### hexo themes/next主题文件存在无法提交github

###### 从暂存区删除该文件夹

```bash
git rm --cache themes/next
```

##### 提交

```bash
git add themes/next/
git commit -m "your description"
git push origin source
```

