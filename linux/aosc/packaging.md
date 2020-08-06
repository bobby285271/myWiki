---
title: AOSC OS Packaging Guide
description: 
published: true
date: 2020-08-06T11:46:26.977Z
tags: guide, linux, packaging, aosc
editor: markdown
---

## AOSC OS 打包踩坑记录

https://wiki.aosc.io/zh/dev-sys-basics

BuildKit 下载的是 [这个](https://mirrors.bfsu.edu.cn/anthon/aosc-os/os-amd64/buildkit/aosc-os_buildkit_latest_amd64.tar.xz)。

**没有执行 `ciel update-os`。**

### `ciel config -i stable` 时提示 `ciel-local.list` 不存在

问题：

```
un-initialize local repository remove stable/etc/apt/sources.list.d/ciel-local.list: no such file or directory
```

解决：在执行这个命令的时候进入 `stable/etc/apt/sources.list.d` 创建一个 `ciel-local.list`。

### 修正以上问题后改提示设备或资源忙

问题：

```
un-initialize local repository OK
                   stop stable --
                unmount stable device or resource busy
```

解决：搁置不管。


### `ciel build -i stable` 提示 `/tree` 不存在

问题：
```
[CRIT]: Oops! FileNotFoundError: [Errno 2] No such file or directory: '/tree'
```


解决：玄学做法。

```
cd stable
mkdir tree
cd ..
cp -r TREE/. stable/tree
```
