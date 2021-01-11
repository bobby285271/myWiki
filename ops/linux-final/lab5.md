---
title: Linux Course Final: Lab 5
description: 
published: true
date: 2021-01-11T04:59:11.935Z
tags: 
editor: markdown
dateCreated: 2021-01-11T04:59:11.935Z
---

## 6 实验五

### 6.1 站点配置

在前面实验三部署 `muban1` 的时候，已经同时准备好了 `muban2` 和 `muban4`，这里不重复相关步骤了。

开两个 VirtualHost，新建并编辑 `/etc/httpd/conf/lab5.conf`：

```
# nano /etc/httpd/conf/lab5.conf
```

```
<VirtualHost *:80>
    ServerAdmin bobbyrong@aosc.io
    DocumentRoot "/var/www/html/muban2"
    ServerName www.traveling.com
    ServerAlias www.traveling.com
    ErrorLog "/var/log/httpd/lab5-error_log"
    CustomLog "/var/log/httpd/lab5-access_log" common
    <Directory "/var/www/html/muban2">
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:80>
    ServerAdmin bobbyrong@aosc.io
    DocumentRoot "/var/www/html/muban4"
    ServerName www.foods.com
    ServerAlias www.foods.com
    ErrorLog "/var/log/httpd/lab5-error_log"
    CustomLog "/var/log/httpd/lab5-access_log" common
    <Directory "/var/www/html/muban4">
        Require all granted
    </Directory>
</VirtualHost>
```

在主配置文件中包含这个文件：

```
# nano /etc/httpd/conf/httpd.conf
```

```
Include conf/lab5.conf
```

校验配置文件合法性，然后重启 httpd：

```
# httpd -t
# systemctl restart httpd
```

### 6.2 站点访问测试

还是在客户端编辑 `/etc/hosts`，和上个实验其实一模一样：

```
192.168.4.5 www.traveling.com
192.168.4.5 www.foods.com
```

在客户端访问这两个网站查看效果。