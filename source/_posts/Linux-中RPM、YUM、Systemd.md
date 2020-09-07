---
title: liunx中RPM、YUM、Systemd
mathjax: false
date: 2020-09-06 23:31:01
categories:
    - Linux
tags:
    - Linux
    - RPM
    - YUM
    - Systemd
---


## RPM软件包

在RPM（红帽软件包管理器）公布之前，要想在Linux系统中安装软件只能采取源码包的方式安装。早期在Linux系统中安装程序是一件非常困难、耗费耐心的事情，而且大多数的服务程序仅仅提供源代码，需要运维人员自行编译代码并解决许多的软件依赖关系，因此要安装好一个服务程序，运维人员需要具备丰富知识、高超的技能，甚至良好的耐心。而且在安装、升级、卸载服务程序时还要考虑到其他程序、库的依赖关系，所以在进行校验、安装、卸载、查询、升级等管理软件操作时难度都非常大。

RPM机制则为解决这些问题而设计的。RPM有点像Windows系统中的控制面板，会建立统一的数据库文件，详细记录软件信息并能够自动分析依赖关系。目前RPM的优势已经被公众所认可，使用范围也已不局限在红帽系统中了。表1-1是一些常用的RPM软件包命令，当前不需要记住它们，大致混个“脸熟”就足够了。

<!-- more -->

表1-1                                                 常用的RPM软件包命令

| 命令                  | 作用                |
| --------------------- | ------------------- |
| rpm -ivh filename.rpm | 安装软件            |
| rpm -Uvh filename.rpm | 升级软件            |
| rpm -e filename.rpm   | 卸载软件            |
| rpm -qpi filename.rpm | 查询软件描述信息    |
| rpm -qpl filename.rpm | 列出软件文件信息    |
| rpm -qf filename      | 查询文件属于哪个RPM |

## **Yum软件仓库**

尽管RPM能够帮助用户查询软件相关的依赖关系，但问题还是要运维人员自己来解决，而有些大型软件可能与数十个程序都有依赖关系，在这种情况下安装软件会是非常痛苦的。Yum软件仓库便是为了进一步降低软件安装难度和复杂度而设计的技术。Yum软件仓库可以根据用户的要求分析出所需软件包及其相关的依赖关系，然后自动从服务器下载软件包并安装到系统。Yum软件仓库的技术拓扑如图1-50所示。

![第1章 部署虚拟环境安装linux系统。第1章 部署虚拟环境安装linux系统。](https://www.linuxprobe.com/wp-content/uploads/2015/01/yum.png)

图1-50  Yum软件仓库的技术拓扑图

Yum软件仓库中的RPM软件包可以是由红帽官方发布的，也可以是第三方发布的，当然也可以是自己编写的。《Linux就该这么学》随书提供的系统镜像（需在书籍站点中网络下载）内已经包含了大量可用的RPM红帽软件包，后文中详细讲解这些软件包。表1-2所示为一些常见的Yum命令，当前只需对它们有一个简单印象即可。

表1-2                                                      常见的Yum命令

| 命令                      | 作用                         |
| ------------------------- | ---------------------------- |
| yum repolist all          | 列出所有仓库                 |
| yum list all              | 列出仓库中所有软件包         |
| yum info 软件包名称       | 查看软件包信息               |
| yum install 软件包名称    | 安装软件包                   |
| yum reinstall 软件包名称  | 重新安装软件包               |
| yum update 软件包名称     | 升级软件包                   |
| yum remove 软件包名称     | 移除软件包                   |
| yum clean all             | 清除所有仓库缓存             |
| yum check-update          | 检查可更新的软件包           |
| yum grouplist             | 查看系统中已经安装的软件包组 |
| yum groupinstall 软件包组 | 安装指定的软件包组           |
| yum groupremove 软件包组  | 移除指定的软件包组           |
| yum groupinfo 软件包组    | 查询指定的软件包组信息       |

## **Systemd初始化进程**

Linux操作系统的开机过程是这样的，即从BIOS开始，然后进入Boot Loader，再加载系统内核，然后内核进行初始化，最后启动初始化进程。初始化进程作为Linux系统的第一个进程，它需要完成Linux系统中相关的初始化工作，为用户提供合适的工作环境。红帽RHEL 7系统已经替换掉了熟悉的初始化进程服务System V init，正式采用全新的systemd初始化进程服务。如果您之前学习的是RHEL 5或RHEL 6系统，可能会不习惯。systemd初始化进程服务采用了并发启动机制，开机速度得到了不小的提升。虽然systemd初始化进程服务具有很多新特性和优势，但目前还是下面4个槽点。

> 槽点1：systemd初始化进程服务的开发人员Lennart Poettering就职于红帽公司，这让其他系统的粉丝很不爽。
>
> 槽点2： systemd初始化进程服务仅仅可在Linux系统下运行，“抛弃”了UNIX系统用户。
>
> 槽点3：systemd接管了诸如syslogd、udev、cgroup等服务的工作，不再甘心只做初始化进程服务。
>
> 槽点4：使用systemd初始化进程服务后，RHEL 7系统变化太大，而相关的参考文档不多，令用户着实为难。

无论怎样，RHEL 7系统选择systemd初始化进程服务已经是一个既定事实，因此也没有了“运行级别”这个概念，Linux系统在启动时要进行大量的初始化工作，比如挂载文件系统和交换分区、启动各类进程服务等，这些都可以看作是一个一个的单元（Unit），systemd用目标（target）代替了System V init中运行级别的概念，这两者的区别如表1-3所示。

表1-3                                   systemd与System V init的区别以及作用

| System V init运行级别 | systemd目标名称   | systemd 目标作用 |
| --------------------- | ----------------- | ---------------- |
| 0                     | poweroff.target   | 关机             |
| 1                     | rescue.target     | 单用户模式       |
| 2                     | multi-user.target | 多用户的文本界面 |
| 3                     | multi-user.target | 多用户的文本界面 |
| 4                     | multi-user.target | 多用户的文本界面 |
| 5                     | graphical.target  | 多用户的图形界面 |
| 6                     | reboot.target     | 重启             |
| emergency             | emergency.target  | 救援模式         |



如果想要将系统默认的运行目标修改为“多用户，无图形”模式，可直接用ln命令把多用户模式目标文件连接到/etc/systemd/system/目录：

```
[root@linuxprobe ~]# ln -sf /lib/systemd/system/multi-user.target /etc/systemd/system/default.target
```

如果有读者之前学习过RHEL 6系统，或者已经习惯使用service、chkconfig等命令来管理系统服务，那么现在就比较郁闷了，因为在RHEL 7系统中是使用systemctl命令来管理服务的。表1-4和表1-5所示RHEL 6系统中System V init命令与RHEL 7系统中systemctl命令的对比，您可以先大致了解一下，后续章节中会经常用到它们。

表1-4            systemctl管理服务的启动、重启、停止、重载、查看状态等常用命令

| 老系统命令          | 新系统命令              | 作用                           |
| ------------------- | ----------------------- | ------------------------------ |
| service foo start   | systemctl start httpd   | 启动服务                       |
| service foo restart | systemctl restart httpd | 重启服务                       |
| service foo stop    | systemctl stop httpd    | 停止服务                       |
| service foo reload  | systemctl reload httpd  | 重新加载配置文件（不终止服务） |
| service foo status  | systemctl status httpd  | 查看服务状态                   |



表1-5    systemctl设置服务开机启动、不启动、查看各级别下服务启动状态等常用命令

| 老系统命令        | 新系统命令                             | 作用                               |
| ----------------- | -------------------------------------- | ---------------------------------- |
| chkconfig foo on  | systemctl enable httpd                 | 开机自动启动                       |
| chkconfig foo off | systemctl disable httpd                | 开机不自动启动                     |
| chkconfig foo     | systemctl is-enabled httpd             | 查看特定服务是否为开机自启动       |
| chkconfig --list  | systemctl list-unit-files --type=httpd | 查看各个级别下服务的启动与禁用情况 |