---
title: SCNUOJ Judge Dev
description: 
published: true
date: 2021-02-08T08:10:33.594Z
tags: scnuoj, online-judge, oj
editor: markdown
dateCreated: 2021-02-08T08:10:33.594Z
---

SCNUOJ 是基于 JNOJ 二次开发的，JNOJ 使用的是 HUSTOJ 魔改过来的判题机。

### 修改编译参数

发现有这样一个文件 `judge/src/language.h`，上面的内容跟编译参数贼像，就是他了。

### 系统调用

当你遇到本地 AC 提交 RE 一定是判题机出现了问题（大雾），这时候就看看检查日志里面是哪个系统调用号，往 `judge/src/okcalls64.h` 加上就行了。当然你可能会有点强迫症，不希望这个文件上一大串不知道是啥的数字，这时候就去 `/usr/include/x86_64-linux-gnu/asm/unistd_64.h` 查一下这个数字，然后再去 `/usr/include/x86_64-linux-gnu/bits/syscall.h` 查一些相应的 define。

当然系统调用也不是随便加的。例如人家 `system("rm -rf /");` 被 RE 那是应该的，这里推荐关注 HUSTOJ 的系统调用设置。

### 记录 AC 测试点数据

你会发现 AC 了的测试点不会显示测试点数据，而且 Web 应用程序也没有方法调，那就可以猜到是判题机的问题了，很快啊你就能找到这样一个函数：

```c
void record_data(problem_struct problem,
                 verdict_struct * verdict_res,
                 char * infile, char * outfile, char * userfile);
```

好家伙，测试数据为啥被截断的原因也顺便找到了。

接下来就看看哪里调用了这个函数：

```c
if (run_result == OJ_AC) {
    pass_count++;
} else {
    // 记录该数据点的运行信息
    record_data(problem, &verdict_res, infile, outfile, userfile);
}
```