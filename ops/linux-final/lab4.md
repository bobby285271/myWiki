---
title: Linux Course Final: Lab 4
description: 
published: true
date: 2021-01-11T04:58:30.476Z
tags: 
editor: markdown
dateCreated: 2021-01-11T04:58:30.476Z
---

## 5 实验四

### 5.1 站点配置

在服务端把 nano 装上了，所以我下面就用 nano 啦。

```
# nano /etc/httpd/conf/httpd.conf
```

找到 `ServerName`，修改为 `rjxy.scnu.cn`。

再找到 `DocumentRoot`，修改为 `/var/ftp`，同时允许访问该目录下的文件。

```
DocumentRoot "/var/ftp"

<Directory "/var/ftp">
    Require all granted
</Directory>
```

### 5.2 PHP 环境配置

把 PHP 环境也给搭建了。

```
# dnf install php-fpm
```

启动 PHP FPM 服务：

```
# systemctl enable php-fpm --now
```

注意到 Apache 这边非常给力，该加载的模块都加载了：

```
# cd /etc/httpd/conf.modules.d
# cat 00-proxy.conf
```

设置默认首页为 `index.php`：

```
# nano /etc/httpd/conf/httpd.conf
```

```
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```

最后重启 httpd：

```
# systemctl restart httpd
```

### 5.3 Host 配置

在服务端 `/var/ftp` 新建一个 `index.php` 文件：

```
# nano /var/ftp/index.php
```

```
<?php phpinfo(); ?>
```

接下来修改客户端的 `/etc/hosts` 文件：

```
# nano /etc/hosts
```

```
192.168.4.5 rjxy.scnu.cn
```

客户端访问 `http://rjxy.scnu.cn/` 查看效果。

### 5.4 站点部署与访问测试

这里部署的是 [SCNU-SoCoding/scnucpc-print-queue-lister](https://github.com/SCNU-SoCoding/scnucpc-print-queue-lister)。

安装 Git：

```
# dnf install git
```

拉取 GitHub 的仓库：

```
# git clone https://github.com/SCNU-SoCoding/scnucpc-print-queue-lister --depth=1 
```

删除测试 PHP 创建的 `index.php`，将仓库文件移动到合适的位置：

```
# rm -rf index.php
# cp -rf scnucpc-print-queue-lister/. ./
# mv .queue.php index.php
# rm -rf scnucpc-print-queue-lister/ .git/
```

这个站点需要 `php-json`，装上：

```
# dnf install php-json
```

由于前面已经配置过 Host，这里直接看效果了，访问 `http://rjxy.scnu.cn/`。