---
title: Linux Course Final: Lab 1
description: 
published: true
date: 2021-01-11T04:56:06.466Z
tags: 
editor: markdown
dateCreated: 2021-01-11T04:56:06.466Z
---

## 2 实验一

### 2.1 vsftpd 安装

下面的命令在服务端执行，把包给装上了先。

```
# dnf search vsftpd
# dnf install vsftpd
```

### 2.2 vsftpd 启动

启动相应的服务：

```
# systemctl enable vsftpd --now
```

### 2.3 ftp 安装

在客户端安装 `ftp`：

```
# dnf install ftp
```

### 2.4 匿名访问配置与测试

由于要允许匿名访问，改一下配置：

```
# vi /etc/vsftpd/vsftpd.conf
```

注释这行：

```
anonymous_enable=NO
```

重启一下服务：

```
# systemctl restart vsftpd
```

按要求创建一下文件：

```
# cd /var/ftp/pub
# vi anon.txt
```

在客户端登录 FTP，用 `anonymous` 登录（密码为空），查看文件。

```
$ ftp 192.168.4.5
> ls
```

### 2.5 用户登录测试

在服务端家目录创建一个 `lisi` 文件夹，然后创建一个 `lisi.txt` 文件：

```
# cd /home/bobby285271/
# mkdir lisi
# cd lisi
# vi lisi.txt
```

在客户端登录 FTP，用 `bobby285271` 登录，查看文件。

```
$ ftp 192.168.4.5
> ls
> cd lisi
> ls
```

### 2.6 wget 获取文件测试

首先安装 `wget` ：

```
# dnf install wget
```

匿名下载 `anon.txt` 文件并验证：

```
$ wget ftp://192.168.4.5/pub/anon.txt
$ cat anon.txt
```

使用用户 `bobby285271` 下载 `lisi.txt`，这里就不直接贴出密码了，前面设了变量：

```
$ wget --ftp-user=bobby285271 --ftp-password=${password} ftp://192.168.4.5/lisi/lisi.txt
$ cat lisi.txt
```

### 2.7 ftp 上传与下载测试

做后面实验的时候用 `wget` 下过一个 `index.html`，正好用上。

首先是使用 `bobby285271` 用户登录 ftp，测试文件上传：

```
# ftp 192.168.4.5
> put index.html
```

删除了本地的测试文件 `index.html`：

```
# rm -f index.html
```

接下来使用 `bobby285271` 用户登录 ftp，测试文件下载。

```
# ftp 192.168.4.5
> get index.html
```

接下来使用匿名用户登录 ftp，测试上传与测试（过程略）。