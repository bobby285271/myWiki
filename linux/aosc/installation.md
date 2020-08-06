---
title: AOSC OS Installation Guide (Lite)
description: 
published: true
date: 2020-08-06T11:42:48.346Z
tags: guide, linux, installation, aosc
editor: markdown
---

给自己看的 AOSC OS 安装指南，参考：https://wiki.aosc.io/en/sys-installation-amd64

## 安装前准备

省略，和 Arch Linux 完全一致。

## 解压 Tarball

```
# cd /mnt
# tar --numeric-owner -pxvf /path/to/tarball/tarball.tar.xz
```

## 生成 /etc/fstab

```
# /mnt/usr/bin/genfstab -U -p /mnt >> /mnt/etc/fstab
```

## 绑定设备和系统路径

```
# mkdir /mnt/run/udev
# for i in dev proc sys run/udev; do mount --rbind /$i /mnt/$i; done
```

## 进入新系统

```
# chroot /mnt /bin/bash
```

## 创建 Init RAM Disk

```
# sh /var/ab/triggered/dracut
```

## 配置引导器

```
# grub-install --target=x86_64-efi --bootloader-id=“AOSC OS" --efi-directory=/boot
# grub-mkconfig -o /boot/grub/grub.cfg
```

## 创建用户

以 `aosc` 为例： 

```
# useradd -m -s /bin/bash aosc
# usermod -a -G audio,cdrom,video,wheel aosc
# chfn -f "AOSC User" aosc
```

## 设置密码

```
# passwd aosc
# passwd root
```

## 设置系统时区

```
# ln -svf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 设置系统语言

编辑 `/etc/locale.conf`：

```
LANG=zh_CN.UTF-8
```

## 设置主机名

编辑 `/etc/hostname`：

```
MyNewComputer
```