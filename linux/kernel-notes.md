---
title: Introduction to Linux OS
description: 
published: true
date: 2020-08-11T10:42:39.846Z
tags: linux, operating-system, kernel
editor: markdown
---

# Linux 学习笔记

自用。更新中。

## 计算机组成原理

资料：[内核完全注释](https://bobby285271.coding.net/p/img/d/img/git/raw/master/Linux%E5%86%85%E6%A0%B8%E8%A7%A3%E6%9E%90.pdf)。

### 系统组成

P17。

* 系统分四部分：输入、输出、处理中心、能源。
* 计算机系统：软件、硬件。

#### 本地总线
* CPU 通过地址线、数据线和控制信号线组成的**本地总线**与系统其他部分进行数据通信。
* 地址线：提供内存和 I/O 设备地址。
* 数据线：CPU 和内存或 I/O 设备数据传输。
* 控制线：执行具体读写操作。

#### 控制器和存储器
* 接口通常都集成在计算机主板上。
* 以一块大规模集成电路芯片为主组成的功能电路。

#### 控制卡（适配器）
* 通过扩展插槽与主板上系统总线连接。

#### 现代
* 南桥（管理低、中速的组件）、北桥（与 CPU、内存和 AGP 视频接口）。