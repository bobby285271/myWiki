---
title: Introduction to Segment Tree (Luogu P3372)
description: 
published: true
date: 2020-08-01T02:50:59.688Z
tags: algorithm, luogu, solution, tree, segment-tree
editor: markdown
---

SCNUACM 新生赛的 C 题被我用 N 方算法水过去了，很惭愧，还是想了解一下正解。当然了，一切的基础，就是熟悉相关的模板咯！

## 题目
https://www.luogu.com.cn/problem/P3372

## 思路
> 以下内容**仅为个人理解，可能存在错误**。

显然，对给出的区间中的每一个数逐个加 k（操作 1）或者一个一个数累加下去来求和（操作 2）是会超时的。我们先抛开操作 1 不谈，试想一下对于操作 2，如果我们可以把这个区间分成几个小块，然后又事先知道每一小块中的元素求和的结果，那我求和的时候是不是就不需要算那么多项呢？是不是也就快一点呢？于是就有了分块思想。如果我们把这个思想给贯彻地彻底一些，事先将几个小块拼在一起形成一个大块，又事先算出每一大块中的元素求和的结果，会不会再更快一些呢？以此类推，树形结构就被我们构建出来了。事实上，线段树就是一种二叉树（但不一定是完全二叉树，下面配图就是一个反例，感谢 CGY 提醒）。它的每一个子节点都表示整个序列中的一段子区间，每个叶子节点都表示序列中的单个元素信息，父节点整合了他的每一个子节点的信息。

> 有人会问既然是求和，直接前缀和不就好了吗。刚开始我也是这样想的，果然后来我就看到了树状数组的模板，后面也会去试一下。但是上面这个思路可以处理更多的情况，例如我求个最大值最小值我也可以应用。

假设我的序列已经读入完毕并储存到数组 a 了（这个简单）：

```cpp
long long a[maxn + 2];
long long n, m;
scanf("%lld%lld", &n, &m);
for (long long i = 1; i <= n; i++)
    scanf("%lld", &a[i]);
```

虽然说不一定是完全二叉数，但是可以用**类似于**创建完全二叉树的方式来创建线段树。首先，考虑单个节点。我们使用一个结构体来储存节点的信息，`l` 和 `r` 是这个节点维护的区间范围，`sum` 储存这个区间的和（也就是说 `sum` 储存了从 `a[l]` 到 `a[r]` 所有元素的和）。这里不妨结合图来理解（转载自洛谷）：

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/xianduanshu.png)

接下来我们考虑整棵数。我们使用数组 t 存这颗树，用数组 t 的下标表示每个节点的编号。为什么数组要开数据量的四倍呢，可以通过计算树的高度，然后用等比数列求和公式得出。

```cpp
struct tree
{
    long long l, r;
    long long sum;
} t[4 * maxn + 2];
```

接下来我们就开始建树。自然要从根开始一层一层地建下去。我们整一个建树函数，考虑向它传入三个参数，其中 `p` 是该节点的编号（其实就是数组 t 的下标），`l` 和 `r` 是这个节点维护的区间范围。

```cpp
void build(long long p, long long l, long long r);
```

我们把这个节点建好了之后就去建它的左右儿子两个节点，显然它们的编号是 `p * 2` 和 `p * 2 + 1`（还是数组 t 的下标），这两个子节点维护的区间范围自然就要减半了。我们假设 `mid = (l + r) / 2`，那么就让左儿子维护 `l` 到 `mid`，右儿子维护 `mid + 1` 到 `r`。

当然，我们不可能一直建下去，当 `l == r` 的时候这个节点只维护一个元素，这个元素正正是 `a[l]`（或者 `a[r]`）。这时就可以开始求 `sum` 而无需再往下建树了。你会发现 `t[p].sum = a[l];`，`p` 依然是这个节点本身的编号。那么那些 `l != r` 的节点又怎样计算 `sum` 呢？就利用上面所说的思想，直接把两个子节点的 `sum` 加在一起就是自己的 `sum` 了。

```cpp
void build(long long p, long long l, long long r)
{
    t[p].l = l;
    t[p].r = r;
    if (l == r)
    {
        t[p].sum = a[l];
        return;
    }
    long long mid = (l + r) / 2;
    build(p * 2, l, mid);
    build(p * 2 + 1, mid + 1, r);
    t[p].sum = t[p * 2].sum + t[p * 2 + 1].sum;
}
```

让我们在 main 函数中调用这个函数，我们让根节点的编号为 1，按照题意根节点维护的区间范围自然是 `l` 到 `n`：

```cpp
build(1, 1, n);
```

好了我们现在再来考虑一下操作 1，操作 1 是否也可以应用类似的思想呢？我们很容易想到一种看起来很简单的方法。同样是将序列分成一个一个的小块，我们直接给整个小块打上一个标签，告诉大家「这个小块里面的元素都应该被加上 k」。而小块中的元素我就干脆不碰它了。于是**懒标签**也就有了。我们在结构体中添加一 `lazy`，用作这个「标签」。

```cpp
struct tree
{
    long long l, r;
    long long sum, lazy;
} t[4 * maxn + 2];
```

假设我现在让 `x` 到 `y` 这个区间的所有元素全部加 z。根据分块思想，我可以拆分这个区间为几个小的区间，使这些小区间的 `sum` 我都在建树的时候提前算过。我假设其中一个小的区间，也就是刚才建好的树的其中一个节点（假设它的编号为 p）维护的区间范围为 `t[p].r` 到 `t[p].l`。我们准备一个函数来处理这件事：

```cpp
void change(long long p, long long x, long long y, long long z);
```

我们直接对 `sum` 加上「元素个数」个 z，然后打上标签，声明 `t[p].r` 到 `t[p].l` 中所有数都应该被加上 z。

```cpp
t[p].sum += z * (t[p].r - t[p].l + 1);
t[p].lazy += z;
```

但是问题也来了，如何正确地**拆分给定区间为几个小的区间**呢。显然我们要让这些**小的区间**的数量尽量少，一个简单粗暴的方法就是：既然我们有一棵树，树上每个节点都有标明它维护的区间范围，我们就从这棵树的根开始，从上往下枚举这些区间，只到找到合适的区间。何为合适？

```cpp
if (x <= t[p].l && y >= t[p].r)
{
    t[p].sum += z * (t[p].r - t[p].l + 1);
    t[p].lazy += z;
}
```

如果条件满足，我们就成功地找到了一个**小的区间**。如果条件不满足，我们还需要继续寻找其它的**小的区间**直到这些小区间拼起来正好是给定的区间。显然一个节点的左儿子和右儿子如果都是上面所说的**合适的**小区间，那么**小的区间**的数量就不是最小了，因为这个节点本身也必然是合适的。于是我们直接考虑下一层的节点。

```cpp
long long mid = (t[p].l + t[p].r) / 2;
if (x <= mid)
    change(p * 2, x, y, z);
if (y > mid)
    change(p * 2 + 1, x, y, z);
```

要注意的是，当子节点的 `sum` 被改变了，要及时地将变更传递回来。

```cpp
t[p].sum = t[p * 2].sum + t[p * 2 + 1].sum;
```

按理来说这个 change 函数我们基本就实现出来了。但是有一个非常坑的地方，就是如果我连续进行多个操作 1，在第一个操作 1 时，我的 `t[p].sum` 恰好是一个合适的小区间，被修改了，但是 `t[p * 2].sum` 和 `t[p * 2 + 1].sum` 是没有变动的，因为变更都被拦截在 `t[p].lazy` 那里了。在第二个操作 1 时，`t[p].sum` 可能不是一个合适的小区间，但 `t[p * 2].sum` 是。`t[p * 2].sum` 被修改了，而且通过上面这一行代码传了回去。试想一下 `t[p].sum` 的值是不是会出现错误呢？于是在必要的时候，懒标记储存的更改还是得下放下去。那懒标记存在的意义到底在哪里？要知道的是我下放懒标记仅仅是因为 `t[p].sum` 不是一个合适的小区间。如果它是的话，这个懒标记还是可以节约时间的。

懒标记的下放很简单，就是将左右儿子的 `sum` 和 `lazy` 给改过来，**把自己的懒标记归零**：

```cpp
void spread(long long p)
{
    if (t[p].lazy)
    {
        t[p * 2].sum += t[p].lazy * (t[p * 2].r - t[p * 2].l + 1);
        t[p * 2 + 1].sum += t[p].lazy * (t[p * 2 + 1].r - t[p * 2 + 1].l + 1);
        t[p * 2].lazy += t[p].lazy;
        t[p * 2 + 1].lazy += t[p].lazy;
        t[p].lazy = 0;
    }
}
```

这样子我们的 change 函数也就完成了！

```cpp
void change(long long p, long long x, long long y, long long z)
{
    if (x <= t[p].l && y >= t[p].r)
    {
        t[p].sum += z * (t[p].r - t[p].l + 1);
        t[p].lazy += z;
        return;
    }
    spread(p);
    long long mid = (t[p].l + t[p].r) / 2;
    if (x <= mid)
        change(p * 2, x, y, z);
    if (y > mid)
        change(p * 2 + 1, x, y, z);
    t[p].sum = t[p * 2].sum + t[p * 2 + 1].sum;
}
```

我们回到开头，我们希望让 `x` 到 `y` 这个区间的所有元素全部加 z，那么我们来调用这个函数。要让这些**小的区间**的数量尽量少，我们从树根开始查找：

```cpp
long long q, x, y, z;
scanf("%lld", &q);
if (q == 1)
{
    scanf("%lld%lld%lld", &x, &y, &z);
    change(1, x, y, z);
}
```

那么如果我想求出给定区间的和，按照上面的思想，同样也是**拆分给定区间为几个小的区间**。我们开一个 `ans` 变量，当找到合适的小区间时就把它的 `sum` 给累加进去。其实和 change 函数的实现几乎一样。但要注意的是，如果发现有懒标记没有下放而这个区间又不是一个合适的小区间的时候，还是要下放。不然如果它的左右孩子有一个是合适的小区间，累加进去的 `sum` 就是错的。

```cpp
long long ask(long long p, long long x, long long y)
{
    if (x <= t[p].l && y >= t[p].r)
        return t[p].sum;
    spread(p);
    long long mid = (t[p].l + t[p].r) / 2;
    long long ans = 0;
    if (x <= mid)
        ans += ask(p * 2, x, y);
    if (y > mid)
        ans += ask(p * 2 + 1, x, y);
    return ans;
}
```

在 main 函数里面调用的时候，还是要从树根开始选取合适的小区间。

```cpp
long long q, x, y;
scanf("%lld", &q);
if (q == 2)
{
    scanf("%lld%lld", &x, &y);
    printf("%lld\n", ask(1, x, y));
}
```

## AC 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const long long maxn = 100010;
long long a[maxn + 2];
struct tree
{
    long long l, r;
    long long sum, lazy;
} t[4 * maxn + 2];

void build(long long p, long long l, long long r)
{
    t[p].l = l;
    t[p].r = r;
    if (l == r)
    {
        t[p].sum = a[l];
        return;
    }
    long long mid = (l + r) / 2;
    build(p * 2, l, mid);
    build(p * 2 + 1, mid + 1, r);
    t[p].sum = t[p * 2].sum + t[p * 2 + 1].sum;
}

void spread(long long p)
{
    if (t[p].lazy)
    {
        t[p * 2].sum += t[p].lazy * (t[p * 2].r - t[p * 2].l + 1);
        t[p * 2 + 1].sum += t[p].lazy * (t[p * 2 + 1].r - t[p * 2 + 1].l + 1);
        t[p * 2].lazy += t[p].lazy;
        t[p * 2 + 1].lazy += t[p].lazy;
        t[p].lazy = 0;
    }
}

void change(long long p, long long x, long long y, long long z)
{
    if (x <= t[p].l && y >= t[p].r)
    {
        t[p].sum += z * (t[p].r - t[p].l + 1);
        t[p].lazy += z;
        return;
    }
    spread(p);
    long long mid = (t[p].l + t[p].r) / 2;
    if (x <= mid)
        change(p * 2, x, y, z);
    if (y > mid)
        change(p * 2 + 1, x, y, z);
    t[p].sum = t[p * 2].sum + t[p * 2 + 1].sum;
}

long long ask(long long p, long long x, long long y)
{
    if (x <= t[p].l && y >= t[p].r)
        return t[p].sum;
    spread(p);
    long long mid = (t[p].l + t[p].r) / 2;
    long long ans = 0;
    if (x <= mid)
        ans += ask(p * 2, x, y);
    if (y > mid)
        ans += ask(p * 2 + 1, x, y);
    return ans;
}

int main()
{
    long long n, m;
    scanf("%lld%lld", &n, &m);
    for (long long i = 1; i <= n; i++)
        scanf("%lld", &a[i]);
    build(1, 1, n);
    for (long long i = 1; i <= m; i++)
    {
        long long q, x, y, z;
        scanf("%lld", &q);
        if (q == 1)
        {
            scanf("%lld%lld%lld", &x, &y, &z);
            change(1, x, y, z);
        }
        else
        {
            scanf("%lld%lld", &x, &y);
            printf("%lld\n",ask(1, x, y));
        }
    }
    return 0;
}
```