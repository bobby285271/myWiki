---
title: Boot Sector Programming (in 16-bit Real Mode)
description: 
published: true
date: 2021-02-08T14:25:29.482Z
tags: os, boot, boot-sect, assembly
editor: markdown
dateCreated: 2021-02-08T14:25:29.482Z
---

## 环境准备

```
pacman -S base-devel nasm qemu
```

工作目录下创建 `boot/boot_sect.asm`，下面的工作都是在这里进行的。

## 死循环

分号开头的是注释，为了让整段程序的大小是 512 个字节，故有 `times 510-($-$$) db 0`。

```asm
loop:
    jmp loop

times 510-($-$$) db 0

;magic number to make BIOS know this is a boot sector
dw 0xaa55
```

转换为机器码：

```
nasm boot/boot_sect.asm -f bin -o boot_sect.bin
```

Qemu 运行：

```
qemu-system-x86_64 boot_sect.bin
```

## 简单的 makefile

```
boot_sect.bin : boot/boot_sect.asm
	nasm boot/boot_sect.asm -f bin -o boot_sect.bin

run : boot_sect.bin
	qemu-system-x86_64 boot_sect.bin

clean : 
	rm -rf boot_sect.bin
```

## hello world

0x10 可以看成 `print(al)`，后面的 `jmp $` 还是死循环。

```
mov ah, 0x0e ;tele-type mode
mov al, 'b'
int 0x10
mov al, 'o'
int 0x10
mov al, 'b'
int 0x10
mov al, 'b'
int 0x10
mov al, 'y'
int 0x10
mov al, '2'
int 0x10
mov al, '8'
int 0x10
mov al, '5'
int 0x10
mov al, '2'
int 0x10
mov al, '7'
int 0x10
mov al, '1'
int 0x10

jmp $ ; jump to current addr

times 510-($-$$) db 0

;magic number to make BIOS know this is a boot sector
dw 0xaa55 
```

## 栈

`mov bp, 0x8000` `mov sp, bp` 设置了栈底：

> Set the base of the stack a little above where BIOS loads our boot sector - so it won’t overwrite us.

```
mov ah, 0x0e ;tele-type mode

mov bp, 0x8000 ;set the base of the stack
mov sp, bp
push 'A'
push 'B'
push 'C'

pop bx
mov al, bl
int 0x10

pop bx
mov al, bl
int 0x10

mov al, [0x7ffe] ;fetch the char at 0x8000 - 0x2 (i.e. 16-bits)

int 0x10 ;print(al)

jmp $ ; jump to current addr

times 510-($-$$) db 0

;magic number to make BIOS know this is a boot sector
dw 0xaa55 
```

等效的代码：

```
stack<char> a;
a.push('A');
a.push('B');
a.push('C');
char b = a.top();
a.pop();
cout << b;

char b = a.top();
a.pop();
cout << b;

cout << a.top();
```