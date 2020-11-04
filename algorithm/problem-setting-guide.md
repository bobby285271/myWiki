---
title: SCNUOJ Problemsetting Guide
description: 
published: true
date: 2020-11-04T11:16:04.161Z
tags: scnuoj, problemsetting
editor: markdown
---

> 本文为华南师范大学软件学院在线判题系统 SCNUOJ 出题指南（助教版）。下面的内容糅杂了编者的一些个人观点，故仅供参考。编写本文时参考到了 OI Wiki 的相关内容。
{.is-info}

## SCNUOJ 判题原理

对于每道程序设计题目，解题思路和代码实现方式都不是唯一的，一个很简单的例子，输出 `SCNUOJ`，在 C++ 下就可以使用下面这些方式：

```cpp
printf("SCNUOJ\n");
puts("SCNUOJ");
std::cout << "SCNUOJ" << std::endl;
```

怎么判断一份代码是否符合题目要求呢?逐份代码人工审查肯定是不可能的，在线判题系统是通过一系列自动化、标准化的测试衡量一份代码是否正确的。SCNUOJ 有两种判题方式，一种叫文本对比，一种叫 Special Judge。但无论是哪种方式，SCNUOJ 都会先编译学生提供的代码，然后反复运行这个程序，通过一系列事先准备好的测试数据验证程序能否给出期望的输出。

例如学生提交了一份 C++ 代码，SCNUOJ 先把这份代码保存为 `Main.cc`，接下来就会编译这份代码：

```
g++ -fno-asm -O2 -Wall -lm --static -std=c++14 -DONLINE_JUDGE -o Main Main.cc
```

> 编译时使用的参数在 https://oj.socoding.cn/wiki/index 有说明。
{.is-info}

编译结束后会得到一个 `Main` 可执行文件。由于我们在上传题目的时候提供了若干组测试数据，也就是 `1.in`、`1.out` 这样的纯文本文件。SCNUOJ 就会利用这些文件测试这个可执行文件：

### 文本比较

关于 Special Judge 是什么请查阅 https://oj.socoding.cn/wiki/spj



## 出题前的准备

### 抱有认真负责的态度
出题是为了给别人做的，本质上是服务他人，让学生在做题时能有所收获，强化程序设计思维，从而达到课程教学目的；而不是与学生较量，劝退学生，逼迫学生转专业。题目质量不过关，不但会影响学生的做题体验，学生疑问多起来最终也是占用自己的时间。

### 自己先尝试动手做题
如果没有接触 OI/ACM 或任何程序设计竞赛，在可能的前提下，在 Codeforces 注册一个账号并尝试完成上面的程序设计题目（不需要是难题，题目 Rating 值小于 1000 的就行，基本就是面向大一新生 AK 杯的那种难度）。Codeforces 是目前最为出名的在线判题系统，题目的上传有着非常严格的把关，近五年的题目在题面排版、数据、输入输出表述方式上是无懈可击的，在做题的过程中可以感受一下。出题思路也非常值得参考。

- Codeforces 游玩攻略 https://www.luogu.com.cn/blog/ezoixx130/codeforces-tutorial
- Codeforces 在线判题系统 http://codeforces.com/

## 题目准备

### 造题系统概览
SCNUOJ 目前有两套造题系统，Polygon System（对所有人开放）和管理员后台自带的造题系统（对管理员和助教开放）。**所有题目都需要在这两套造题系统中准备。** 两者各有优缺点，使用起来都非常容易上手，大家目前可以根据自己的需要进行选择，我们会在后期逐步合并两个造题系统的功能。

- Polygon System https://oj.socoding.cn/polygon
- 管理员后台自带的造题系统 https://oj.socoding.cn/admin/problem/index

下面是 Polygon System 提供的一些特性：

- 支持上传标程代码后利用标程代码自动生成测试数据的 `*.out` 文件。

> 在 Polygon System 造的题目需要先同步到管理员后台自带的造题系统才能在 SCNUOJ 中使用。同步方式就是访问 https://oj.socoding.cn/admin/problem/create-from-polygon 并输入 Polygon System 的题目 ID。
{.is-warning}

下面是管理员后台自带的造题系统提供的一些特性：

- 支持 Special Judge 题目的验题。

在完成造题（如果在 Polygon System 完成造题后要先完成同步）之后，即可直接在小组作业或比赛中添加造好的题目。如果你希望将题目添加到公共题库（特指 https://oj.socoding.cn/problem/index 能看到的题目）或者添加到公共比赛（特指 https://oj.socoding.cn/contest/index 能看到的比赛），请联系任意任一香农先修班负责人。

### Polygon System 使用流程

- 在 https://oj.socoding.cn/ 注册一个账户并登录。
- 顶部导航栏 - 

### 题目内容
在 SCNUOJ 上布置的题目，主要目的应当是培养程序设计思维，让学生在见到程序设计类题目时可以有具体的实现想法，能将题面转化为逻辑化的程序语言，次之才是语法学习：例如题目是读入两个数后比较它们的大小，学生应该能马上意识到要声明两个变量，将读入的数字保存到变量，用分支语句比较两个变量的值，输出结果；鼓励在满足时空要求的前提下让学生做到一题多解，除非是希望学生刻意学习、刻意练习，非必要情况下不应当在语法上做过多限制，例如只能利用指针进行某个操作、必须声明一个什么函数等等。

题目可以来源课本和其它在线判题系统，也可以自己原创。特别地，对于《C 语言程序设计》课本原题，这些题目往往不是专门为 OJ 设计的，直接照搬请注意移植到 SCNUOJ 之前可能需要做一定的修改，详见下面章节的介绍；对于原创题，注意题目中不应过多涉及到其它学科的知识，如果涉及应当给予详细的解释，不应使其它学科的知识作为解题的重大障碍；思维难度应占主要部分，避免强行拼凑多个题目、条件或知识点而达到降低 AC 人数的目的。
