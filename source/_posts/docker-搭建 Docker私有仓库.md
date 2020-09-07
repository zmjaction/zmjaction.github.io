
---
title: 搭建 Docker私有仓库
mathjax: false
date: 2018-03-12 10:17:51
categories:
    - docker
    - repositories
tags:
    - docker
    - repositories
---
#### Registry部署
```go
docker search registry
docker pull registry
```
然后启动一个容器，这里的 /opt/registry 是我们本地的目录，用于存储上传的镜象，/var/lib/registry是Registry服务默认的保存镜象目录
```go
docker run -d -v /opt/registry:/var/lib/registry -p 5000:5000 --restart=always --name registry registry
```
运行 docker ps 看一下容器情况，

<!-- more -->
```
docker ps -a

9ea93421fc13        registry            "/entrypoint.sh /etc…"   27 minutes ago      Up 11 minutes                 0.0.0.0:5000->5000/tcp   registry

```
说明我们已经启动了registry服务，打开浏览器输入http://主机ip:5000/v2，正常返回如下数据
```
{}
```
注意：在/etc/docker/daemon.json中需要添加,不然推送不成功
```
{ "insecure-registries":["192.168.1.71:5000"] }
```
#### 验证
现在我们通过将镜像push到registry来验证一下。

我的机器上有个hello-world的镜像，我们要通过docker tag将该镜像标志为要推送到私有仓库，
```
docker tag hello-world 127.0.0.1:5000/hello-world
```
然后查看以下本地的镜像
```
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
192.168.1.71:5000/hello-world   v1.1                bf756fb1ae65        7 months ago        13.3kB
hello-world                     latest              bf756fb1ae65        7 months ago        13.3kB

```
接下来，我们运行docker push将hello-world镜像push到我们的私有仓库中，

```go
 docker push 192.168.1.71:5000/hello-world:v1.1 

The push refers to repository [192.168.1.71:5000/hello-world]
9c27e219663c: Pushed 
v1.1: digest: sha256:90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042 size: 525

```
现在我们可以查看我们本地/opt/registry目录下已经有了刚推送上来的hello-world。我们也在浏览器中输入http://192.168.1.71:5000/v2/_catalog，正常返回如下数据
现在我们可以先将我们本地的127.0.0.1:5000/hello-world和hello-world先删除掉，
```
docker rmi hello-world
docker rmi 127.0.0.1:5000/hello-world
```
然后使用docker pull从我们的私有仓库中获取hello-world镜像

```
docker pull 127.0.0.1:5000/hello-world
```