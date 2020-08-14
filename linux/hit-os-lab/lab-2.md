---
title: HIT Operating System Course: Lab 2 Writeup
description: 
published: true
date: 2020-08-14T14:31:22.039Z
tags: linux, operating-system, lab, hit-os-lab
editor: markdown
---

# Lab 2 系统引导

## 修改启动时提示

```
nano boot/bootsect.s
```

把 `mov $24, %cx` 修改为字符串长度（预留三个换行加回车，六个字符）

```
mov     $17, %cx
mov     $0x0007, %bx            # page 0, attribute 7 (normal)
#lea    msg1, %bp
mov     $msg1, %bp
mov     $0x1301, %ax            # write string, move cursor
int     $0x10
```

自定义信息：

```
msg1:
        .byte 13,10
        .ascii "bobby285271"
        .byte 13,10,13,10

        .org 508
```

## 裁剪 bootsect



## Key

https://blog.csdn.net/wangjianyu0115/article/details/46853093