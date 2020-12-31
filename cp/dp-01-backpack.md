---
title: Dynamic Prgramming - 0/1 Backpack Problem (Luogu P1164, P1048, P1020)
description: 
published: true
date: 2020-08-01T02:55:04.834Z
tags: algorithm, luogu, dp, solution, greedy
editor: markdown
---

为啥突然想要看动态规划呢，是因为昨天做了去年蓝桥杯预选赛 16/17 的题，做到 B 题发现正解其实应该是动态规划的。虽然我交了一发贪心过了，但想到两个星期后就是热身赛还是得搞搞突击才行。

## 引入
周四香农先修班讲了一个斐波拉契数列的专题，有一个问题是这样的：假设有 n 级楼梯，我从底部往上爬，每次可以上 1 级，也可以上 2 级楼梯，问从底部到顶部一共有多少种爬楼梯的方法。

我第一时间想到的是搜索和回溯，但是既然是斐波拉契数列的专题，也就有了 $f(n) = f(n-1) + f(n-2)$ 这个公式。我简单说一下这个式子是怎么来的吧。我们不妨假设我们走了 x 步之后上到了第 n 级，那我第 x-1 步在什么位置呢？只能是第 n-1 和 n-2 级。$f(n-1)$ 是走到 n-1 级楼梯时的方法数，$f(n-2)$ 同理。考虑一下边界情况，就是斐波拉契数列了呗。

## P1164 -（二维 DP）
题目：https://www.luogu.org/problem/P1164

不妨就来考虑一下「引入」小节的思路（模拟退火是啥我也不会），我们要求吃够 m 块钱，和爬 n 级楼梯其实是一回事。我**上 1 级台阶还是 2 级台阶**，其实和对于一道菜，我**吃还是不吃**是一回事。区别在哪呢，这里我们有很多不同的菜。然而我们也不用想太多，就让它们按顺序逐个上菜好了，然后我们**逐个**决定是吃还是不吃，顺便记下到底花了多少钱~~（注意上菜不代表我们就要吃嘛对吧，吃了才花钱）~~。

我们定义 `f[i][j]` 为上了 i 道菜用光 j 元钱的办法总数。假设我们现在已经上了 i-1 道菜，接下来上第 i 道菜。我们就有两个选择：

* 吃（花钱）。
* 不吃（不花钱）。

如果不吃，那我上 i 道菜用光 j 元钱的方法数，和上了 i-1 道菜用光 j 元钱的方法数一样。如果我吃，那我上 i 道菜用光 j 元钱的方法数，和上了 i-1 道菜用光（j-这道菜的价格）的方法数一样。将两种情况和在一起，于是有：

```cpp
f[i][j] = f[i - 1][j] + f[i - 1][j - 第i道菜的价格];
```

通常来讲这就结束了，但是这道菜的价格我们却没有讨论清楚。首先，如果我没钱吃着道菜呢？那就只能不吃了呗。


```cpp
if(j < 第i道菜的价格) f[i][j] = f[i - 1][j];
if(j >= 第i道菜的价格) f[i][j] = f[i - 1][j] + f[i - 1][j - 第i道菜的价格];
```

有些人想问 `j == 第i道菜的价格` 这种情况怎么搞，我们将这种情况代入到上面的代码，其实就是 `f[i][j] = f[i - 1][j] + f[i - 1][0]`，`f[i - 1][0]` 其实就是上了 i-1 个菜我全不吃，消费 0 元嘛，显然只有一种情况。但形如 `f[i][0]` 的数据又可以从哪推出呢？不如特判一下，当然直接给他们赋值也是可以的：

```cpp
if(j < 第i道菜的价格) f[i][j] = f[i - 1][j];
if(j == 第i道菜的价格) f[i][j] = f[i - 1][j] + 1;
if(j > 第i道菜的价格) f[i][j] = f[i - 1][j] + f[i - 1][j - 第i道菜的价格];
```

既然有了递推式（状态转移方程），我们要的是上完 n 道菜后花掉 m 块钱的方案，那就遍历一下这个数组直到得出答案为止呗。上代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
int a[101], f[101][10001];
int main()
{
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
    }
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            if (j > a[i])
                f[i][j] = f[i - 1][j] + f[i - 1][j - a[i]];
            if (j == a[i])
                f[i][j] = f[i - 1][j] + 1;
            if (j < a[i])
                f[i][j] = f[i - 1][j];
        }
    }
    cout << f[n][m] << endl;
    return 0;
}
```

## P1048（一维 DP）

题目：https://www.luogu.org/problem/P1048

这道题按照上面的思路打大概是怎样的呢？~~显然~~，我们开一个二位数组 `f[i][j]` 来表示过目 i 件药材（过目意味着我在采摘和不采摘中二选一）耗去 j 时间后的最大价值就可以了。时间如果足够，就考虑一下要不要采摘这个药材。同样地，**采摘**第 i 件药材耗去 j 时间的这种情况下的最大价值是过目 i-1 件药材时耗去（j-药材采摘时间）时间的最大价值。**不采摘**第 i 件药材耗去 j 时间的这种情况下的最大价值是过目 i-1 件药材时耗去 j 时间的最大价值。对于两种情况我们取较大者即可。

那我们为啥提出希望降维呢？我们看上面的代码：

```cpp
if (j > a[i])
    f[i][j] = f[i - 1][j] + f[i - 1][j - a[i]];
if (j == a[i])
    f[i][j] = f[i - 1][j] + 1;
if (j < a[i])
    f[i][j] = f[i - 1][j];
```

再来看一下这一题的核心代码（`a[i]` 为采摘时间，`v[i]` 为药材价值）：

```cpp
if (j >= a[i])
    f[i][j] = max(f[i - 1][j - a[i]] + v[i], f[i - 1][j]);
else
    f[i][j] = f[i - 1][j];
```

第 i 件物品需要求的信息永远是从第 i-1 件物品那里得出来的。如果我们最后只需要第 i 件物品的答案而不需要 i-1、i-2... 件物品的答案，我们为何不用新的信息覆盖旧的信息呢？修改的方法很简单：

```cpp
if (j >= a[i])
    f[j] = max(f[j - a[i]] + v[i], f[j]);
// 下面的没必要保留了吧。
// else
//     f[j] = f[j];
```

虽然降维了，但遍历还是要以上面的方式来，于是我们就得到一个似乎很有道理的代码。

```cpp
for (int i = 1; i <= m; i++)
{
    for (int j = a[i]; j <= t; j++) // 既然上面只有一个分支结构，那我在循环条件那里保证就好了。
    {
        f[j] = max(f[j - a[i]] + v[i], f[j]);
    }
}
cout << f[t] << endl;
```

这却是一个非常典型的错误，因为按照原本的意思，第 i 个物品的数据要从 i-1 个物品的数据推出来，但我在上面求第 i 个物品的 `f[j]` 的时候却使用到了已经更新过的，已经是第 i 个物品的数据 f[j - a[i]]。怎样避免呢：

```cpp
#include <bits/stdc++.h>
using namespace std;
int main()
{
    int t, m;
    cin >> t >> m;
    int a[m + 10], v[m + 10];
    for (int i = 1; i <= m; i++)
    {
        cin >> a[i] >> v[i];
    }
    int f[t + 10];
    memset(f, 0, sizeof(f));
    for (int i = 1; i <= m; i++)
    {
        for (int j = t; j >= a[i]; j--)
        {
            f[j] = max(f[j], f[j - a[i]] + v[i]);
        }
    }
    cout << f[t] << endl;
    return 0;
}
```

附：[一位神犇提供的题解，感觉比较易懂](https://www.luogu.org/blog/lenfrey/post-ti-xie-p1048-cai-yao)

## P1020（最大上升子序列）

题目：https://www.luogu.com.cn/problem/P1020

第二问知道是求最大上升序列之后就很简单了，最大的问题在于为什么是求这个。试给出以下解释（严格证明需要组合数学的知识）：对于给出的序列（输入数据），必定存在至少一个的最大上升子序列。任取一个最大上升子序列，其中任两个元素（导弹）必定由两套不同系统进行拦截。意味着如果最大上升子序列长度为 n，我们最少需要的系统数目必定大于或等于 n。
接下来可以用数学归纳法去解释为什么当最大上升子序列长度为 n 时最少只需要 n 套系统。
当 n = 1 时，给出的序列（输入数据）是单调递减的，显然只需要一套系统，结论成立。
假设 n = k 时结论成立。也就是只需要 k 套系统。当 n = k + 1 时，试将给出的序列一分为二分别求解，具体方法如下：枚举出所有的最大上升子序列，将每个最大上升子序列的尾元素分别取出。如果某个数是多个最大上升子序列的尾元素则只取出一次。将取出的数按照原本的先后顺序排列生成一个新的序列（假设叫序列 1）。未取出的数也按照原来的先后顺序排列形成另一个新的序列（假设叫序列 2）。
此时序列 2 的最大上升子序列长度必定为 k，那么根据前面的假设，需要 k 套系统。而序列 1 是单调递减的（可用反证法证明），由已知需要 1 套系统。所以加起来就是 k + 1 套系统。因为 k + 1 套系统可以拦截所有导弹，而由已知，需要配备的系统数需要大于等于 k + 1，所以 n = k + 1 时结论也成立。

100 分代码（n 方算法）
```cpp
#include <bits/stdc++.h>
using namespace std;

int a[100005], num = 1;
int ans[100005];
int main()
{
    while (cin >> a[num])
        num++;
    num--;
    a[0] = 50001;
    int maxans = 0;
    for (int i = 1; i <= num; i++)
    {
        for (int j = 0; j < i; j++)
        {
            if (a[i] <= a[j])
            {
                // cout << ans[i] << " " << ans[j] + 1 << endl;
                ans[i] = max(ans[i], ans[j] + 1);
            }
        }
        // cout << ans[i] << endl;
        if (ans[i] > maxans)
            maxans = ans[i];
    }
    cout << maxans << endl;

    memset(ans,0,sizeof(ans));
    maxans = 0;
    a[0] = 0;
    for (int i = 1; i <= num; i++)
    {
        for (int j = 0; j < i; j++)
        {
            if (a[i] > a[j])
            {
                // cout << ans[i] << " " << ans[j] + 1 << endl;
                ans[i] = max(ans[i], ans[j] + 1);
            }
        }
        // cout << ans[i] << endl;
        if (ans[i] > maxans)
            maxans = ans[i];
    }
    cout << maxans << endl;
    return 0;
}
```

200 分代码（nlogn 算法）
```
#include <bits/stdc++.h>
using namespace std;
int a[100010], b[100010];
int num = 0, p, ans;

bool cmp(int a, int b)
{
    return a > b;
}

int main()
{
    while (cin >> a[num])
        num++;

    ans = 1;
    b[0] = a[0];
    for (int i = 1; i < num; i++)
    {
        if (a[i] <= b[ans - 1]){
            b[ans] = a[i];
            ans++;
        }
        else
        {
            p = upper_bound(b, b + ans, a[i], cmp) - b;
            b[p] = a[i];
        }
    }
    // for (int i = 0; i < ans; i++)
    // {
    //     cout << b[i] << " ";
    // }
    // cout << endl;
    cout << ans << endl;

    ans = 1;
    b[0] = a[0];
    for (int i = 1; i < num; i++)
    {
        if (a[i] > b[ans - 1]){
            b[ans] = a[i];
            ans++;
        }
        else
        {
            p = lower_bound(b, b + ans, a[i]) - b;
            b[p] = a[i];
        }
    }
    // for (int i = 0; i < ans; i++)
    // {
    //     cout << b[i] << " ";
    // }
    // cout << endl;
    cout << ans << endl;
    return 0;
}
```