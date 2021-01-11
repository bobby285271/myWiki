---
title: Linux Course Final: Lab 3
description: 
published: true
date: 2021-01-11T04:57:54.958Z
tags: 
editor: markdown
dateCreated: 2021-01-11T04:57:54.958Z
---

## 4 实验三

### 4.1 httpd 安装

实验开始前停止实验二开启的有关服务。

根据题目要求安装相应的软件包。

```
# dnf install httpd httpd-manual
```

### 4.2 httpd 启动

启动 httpd 服务并确认当前状态。

```
# systemctl enable httpd --now
# systemctl status httpd
```

### 4.3 站点访问测试

实验二时已经为客户端安装 Lynx 浏览器，这里省略相关步骤。

依然是按下 `G` 编辑地址 `http://192.168.4.5/`，回车访问。

查看 httpd 文档相应的配置文件，得知文档访问方式：

```
# cat /etc/httpd/conf.d/manual.conf
```

按下 `G` 编辑地址 `http://192.168.4.5/manual/`，回车访问。

### 4.4 站点首页自定义

这里部署软协服务器的导航页。首先安装 OpenSSH：

```
# dnf install openssh openssh-clients
```

直接拷贝导航页目录的所有非隐藏文件过来（就不用拷贝 `.git` 目录了）：

```
# scp -r scnuoj@10.191.65.243:/var/www/html/* ./
```

访问 `http://192.168.4.5/` 查看效果。

### 4.5 站点子目录部署

从励儒云下载了模板压缩包。从宿主机移动至 Podman 容器：

```
# podman cp ./test_web.zip server-rongjialin:/var/www/html
```

解压文件：

```
# unzip test_web.zip
```

访问 `http://192.168.4.5/muban1/` 查看效果。