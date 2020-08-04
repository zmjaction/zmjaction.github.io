---
title: docker -- Hub -- 镜像加速器
mathjax: false
date: 2020-05-12 21:21:30
categories:
    - docker
    - Hub镜像
tags:
    - docker
    - Hub镜像
---
## Docker Hub 镜像加速器列表

镜像加速器 | 镜像加速器地址 | 专属加速器[？](# "需登录后获取平台分配的专属加速器") | 其它加速[？](# "支持哪些镜像来源的镜像加速")
--- | --- | --- | --
~~[Docker 中国官方镜像](https://docker-cn.com/registry-mirror)~~ | ~~`https://registry.docker-cn.com`~~ | | ~~Docker Hub~~（[已关闭](https://github.com/docker/docker.github.io/issues/8793)）
[DaoCloud 镜像站](https://daocloud.io/mirror) | `http://f1361db2.m.daocloud.io` | 可登录，系统分配 | Docker Hub
[Azure 中国镜像](https://github.com/Azure/container-service-for-azure-china/blob/master/aks/README.md#22-container-registry-proxy) | `https://dockerhub.azk8s.cn` | | Docker Hub、GCR、Quay
[科大镜像站](https://mirrors.ustc.edu.cn/help/dockerhub.html) | `https://docker.mirrors.ustc.edu.cn` | | Docker Hub、[GCR](https://github.com/ustclug/mirrorrequest/issues/91)、[Quay](https://github.com/ustclug/mirrorrequest/issues/135)
[阿里云](https://cr.console.aliyun.com) | `https://<your_code>.mirror.aliyuncs.com` | 需登录，系统分配 | Docker Hub
[七牛云](https://kirk-enterprise.github.io/hub-docs/#/user-guide/mirror) | `https://reg-mirror.qiniu.com` | | Docker Hub、GCR、Quay
[网易云](https://c.163yun.com/hub) | `https://hub-mirror.c.163.com` | | Docker Hub
[腾讯云](https://cloud.tencent.com/document/product/457/9113) | `https://mirror.ccs.tencentyun.com` | | Docker Hub
由于镜像服务可能出现宕机，建议同时配置多个镜像。各个镜像站测试结果请到 docker-practice/docker-registry-cn-mirror-test 查看。
<!-- more -->
## 配置加速地址
> Ubuntu 16.04+、Debian 8+、CentOS 7
创建或修改 `/etc/docker/daemon.json`：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://1nj0zren.mirror.aliyuncs.com",
        "https://docker.mirrors.ustc.edu.cn",
        "http://f1361db2.m.daocloud.io",
        "https://dockerhub.azk8s.cn"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
### Windows 10
对于使用 Windows 10 的用户，在任务栏托盘 Docker 图标内右键菜单选择 Settings，打开配置窗口后在左侧导航菜单选择 Docker Engine，在右侧像下边一样编辑 json 文件，之后点击 Apply & Restart 保存后 Docker 就会重启并应用配置的镜像地址了。

```
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```

### macOS
对于使用 macOS 的用户，在任务栏点击 Docker Desktop 应用图标 -> Perferences，在左侧导航菜单选择 Docker Engine，在右侧像下边一样编辑 json 文件。修改完成之后，点击 Apply & Restart 按钮，Docker 就会重启并应用配置的镜像地址了。
```
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```
## 检查加速器是否生效

执行 `$ docker info`，如果从结果中看到了如下内容，说明配置成功。

```bash
Registry Mirrors:
 https://hub-mirror.c.163.com/
```

## `k8s.gcr.io` 镜像

可以登录 [阿里云 容器镜像服务](https://www.aliyun.com/product/acr?source=5176.11533457&userCode=8lx5zmtu&type=copy) **镜像中心** -> **镜像搜索** 查找。

例如 `k8s.gcr.io/coredns:1.6.7` 镜像可以用 `registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.6.7` 代替。

一般情况下有如下对应关系：

```bash
# $ docker pull k8s.gcr.io/xxx

$ docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/xxx
```
参考：
> https://gist.github.com/y0ngb1n/7e8f16af3242c7815e7ca2f0833d3ea6
> https://github.com/yeasy/docker_practice/blob/master/install/mirror.md
> https://www.ilanni.com/?p=14534