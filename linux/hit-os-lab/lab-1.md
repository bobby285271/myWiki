---
title: HIT Operating System Course: Lab 1 Writeup
description: 
published: true
date: 2020-08-14T03:36:07.076Z
tags: linux, operating-system, lab, hit-os-lab
editor: markdown
---

# Lab 1 起始课

希望在本地实验，使用了 [tinylab](http://tinylab.org/build-linux-0-11-lab-with-docker/)。

克隆实验环境：

```
git clone https://gitee.com/tinylab/cloud-lab.git
```

初始化并进入容器：

```
cd cloud-lab
tools/docker/run linux-0.11-lab
```

修改软件源：

```
vi /etc/apt/sources.list
apt update
```
```
deb http://mirrors.bfsu.edu.cn/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.bfsu.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.bfsu.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse
deb http://mirrors.bfsu.edu.cn/ubuntu/ trusty-security main restricted universe multiverse
```

Linux 0.11 源码位于 `/labs/linux-0.11-lab/0.11`。

调试：

```
make start-fd
```