---
title: Linux Course Final: Preparation
description: 
published: true
date: 2021-01-11T04:55:19.632Z
tags: 
editor: markdown
dateCreated: 2021-01-11T04:55:19.632Z
---

## 1 前序准备工作

### 1.1 Podman 准备

采用 Podman 容器进行演示（配置略），验证安装：

```
$ pacman -Qs podman
```

从 DockerHub 拉取 CentOS 的镜像：

```
# podman pull centos
```

### 1.2 网络配置与容器创建

为了方便容器间网络通信，创建新的 Bridge 网络，指定一下网段：

```
# podman network create --subnet=192.168.4.0/24 final-rongjialin
```

创建两个实例，`rongjialin-client`  为客户端，`rongjialin-server` 为服务端，这里指定了 IP：

```
# podman run -it --name client-rongjialin --network final-rongjialin --ip 192.168.4.205 centos /bin/bash
# podman run -it --name server-rongjialin --network final-rongjialin --ip 192.168.4.5 centos /bin/bash
```

验证容器是否创建成功：

```
# podman ps -a
```

现在启动刚才创建的两个实例并进入：

```
# podman exec -it server-rongjialin bash
# podman exec -it client-rongjialin bash
```

查看本机 IP 地址：

```
# ip address
```

互相 PING 测试：

```
# ping -c 3 ping -c 3 192.168.4.205
# ping -c 3 ping -c 3 192.168.4.5
```

### 1.3 系统升级

看一下拉下来的是 CentOS Linux 8 还是 CentOS Stream 8。

```
# cat /etc/os-release
```

参考 [USTC Mirror 文档](https://mirrors.ustc.edu.cn/help/centos.html)，把软件源换掉。

```
# sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-Linux-AppStream.repo \
         /etc/yum.repos.d/CentOS-Linux-BaseOS.repo \
         /etc/yum.repos.d/CentOS-Linux-Extras.repo \
         /etc/yum.repos.d/CentOS-Linux-PowerTools.repo \
         /etc/yum.repos.d/CentOS-Linux-Plus.repo
```

更新一下系统：

```
# dnf distro-sync
```

### 1.4 用户创建

创建一个普通用户，用于后面的测试：

```
# useradd -m -g users -G wheel -s /bin/bash bobby285271
# passwd bobby285271
```
