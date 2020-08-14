---
title: HIT Operating System Course: Lab 1 Writeup
description: 
published: true
date: 2020-08-14T03:03:35.855Z
tags: linux, operating-system, lab, hit-os-lab
editor: markdown
---

## Lab 1 起始课

bochs：x86 PC 机模拟器。

解压实验包：

```
tar -zxvf hit-oslab-linux-20110823.tar.gz
```

编译内核：

```
make clean #optional，删除上次编译生成的中间文件和目标文件
cd linux-0.11
make all #会自动跳过未修改的文件， -j 参数并行编译（如 -j 2）
```

在 bochs 运行编译好的内核：`./run`

汇编级调试：`./dbg-asm`，`help` 查看帮助

C 语言级调试：`./dbg-c` 然后 `./rungdb`

访问 `hdc-0.11-new.img`：挂载 `sudo ./mount-hdc`，卸载 `sudo umount hdc`

！`./run` 的时候不要挂载。
