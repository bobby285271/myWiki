---
title: SCNUCPC 2020
description: 
published: true
date: 2021-02-08T08:06:39.672Z
tags: scnu, scnucpc, 2020, editorial
editor: markdown
dateCreated: 2021-02-08T08:06:39.672Z
---

# A. Chino's homework

## 思路

三进制状态压缩动态规划。对每个行而言，每个格子有 $3$ 种状态：

* $0$ 代表不放棋子。
* $1$ 代表 Chino 的棋子。
* $2$ 代表 Cocoa 的棋子。

那么每行就有 $3^m$ 个状态。

设 $f[n][u][v][state]$ 表示放完前 $n$ 行，Chino 用了 $u$ 个棋子，Cocoa 用了 $v$ 个棋子，第 $m$ 行的状态时，所能获得的分数最大值。

此处假设 $count(b,st)$ 是状态 $st$ 下有多少个 $b$ 。那么：

$$
f[n][u][v][state] = \max_{state'}f[n-1][u-count(1,state)][v-count(2,state)][state']+cost[state'][state]
$$

注意只有满足下面的条件时才可以转移：

$$
u \geq count(1,state) \wedge v\geq count(2,state)
$$

先处理好行内（列相邻） $3^m$ 个状态的分数，再处理一个行转移到另外一个行的状态（行相邻）的分数，就可以 $O(m\cdot3^{2m})$ 构造完 $cost$ 数组。

计算方案数时，直接在转移的时候算好：如果新状态比之前更优，则直接把之前的状态数覆盖。如果新状态和之前一样优，那么直接把方案数相加。

另外我们题目不要求棋子用完，所以还需要对 $u,v$ 求一个前缀和，暴力即可。



# B. Query

## 思路

先考虑如下问题：在多个数中选择一个数，使得与给定的 $k$ ，异或后的数值最大。此问题可以通过如下方法解决， 先把所有的数转化为二进制，再插入 Trie（字典树）中，随后查询 $k$ 时，从 Trie 的根节点出发，贪心地优先选择与 $k$ 的二进制数位相反的路径， 这样得到的异或数值将会最大。

本问题需要对二维数组中的子数组进行操作，所以需要二维的区间范围查找，这可以使用二维线段树。我们可以建立一棵二维线段树， 每个线段树的节点存放一棵 Tire， Trie 中存放该子矩阵中所有数， 这样， 建树的时间复杂度为 $O(n^2 log^2 n \ log a )$ 。 对于单次查询，可以将子矩阵划分成 $O(log ^ 2 n)$ 个更小的矩阵， 随后， 对每一个 Tire 都进行查询， 然后合并答案。 这里每次查询需要 $O(log k)$ 的时间， 所以单词查询的时间复杂度为 $O(log^2 n \ log k) $ ， 由于我们需要查询的是下标， 所以， 我们在 Trie 的末尾处标记下标即可。 

总的时间复杂度为 $ O (n^2 log ^2 n \ log a  + n log ^2 n \ log k ) $ 。



# C. Paint

## 题意

题意的意思是给出 $n \times 3$ 的网格，先要给每个格子染**红、黄、绿**三种颜色，要求相邻的格子颜色不能相同，问有多少种染色方案，方案数可能很大，模 $10 ^ 9 + 7$ 后输出。

## 思路

先从 $1 \times 3$ 网格开始考虑，显而易见 $1 \times 3$ 的网格的染色方案有 $12$ 种。

这 $12$ 种方案可以分为两种染色方案：

* $aba$ - 左右相同中间不同。
* $abc$ - 左中右各不相同。

开始讨论当前层的 $aba$ 或 $abc$ 方案可以对下一层产生多少种 $aba$ 和 $abc$ 。
  - $aba$ - 一种 $aba$ 方案可以产生 $3$ 种 $aba$ 和 $2$ 种 $abc$ 方案。
  - $abc$ - 一种 $abc$ 方案可以产生 $2$ 种 $aba$ 和 $2$ 种 $abc$ 方案。

最后根据规律，写出动态规划状态转移方程：

$$
dp_{aba}[i] = (3 \times dp_{aba}[i - 1] + 2 \times dp_{abc}[i - 1]) \bmod (10^9+7) \\
dp_{abc}[i] = (2 \times dp_{aba}[i - 1] + 2 \times dp_{abc}[i - 1])  \bmod (10^9+7)
$$

## 参考代码

```c++
#include <iostream>
using namespace std;
typedef long long ll;
const ll MOD = 1e9 + 7;

int main()
{
    int n;
    cin >> n;
    ll aba = 6;
    ll abc = 6;
    for (int i = 2; i <= n; ++i)
    {
        ll newaba = 3 * aba + 2 * abc;
        ll newabc = 2 * aba + 2 * abc;
        aba = newaba % MOD;
        abc = newabc % MOD;
    }
    cout << ((aba + abc) % MOD);
    return 0;
}
```

# D. Hacker Elitedj

## 题意

求解斐波那契数列前 $n$ 项的平方和，$n(1 \leq n \leq 10^{18})$ ，答案很大，需要模 $10^9+7$ 。

## 思路

矩阵快速幂模板题。首先你要知道这个式子（ $F$ 和 $f$ 的定义见题意）:

$$
F(n) = f(n) \times f(n + 1)
$$ 

题目的 $n$ 很大，线性递推求解斐波那契数列第 $n$ 项肯定是不行的，所以我们需要使用矩阵快速幂。

公式如下：

$$
\begin{bmatrix}f(n+1) \\ f(n)\end{bmatrix} = 
\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix} *
\begin{bmatrix}f(n) \\ f(n - 1)\end{bmatrix}
$$

推导出：

$$
\begin{bmatrix}f(n+1) \\ f(n)\end{bmatrix} = 
\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}^{n} *
\begin{bmatrix}f(1) \\ f(0)\end{bmatrix}
$$

$$
\begin{bmatrix}f(n+1) & f(n) \\ f(n) & f(n - 1) \end{bmatrix} = 
\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}^{n-1} *
\begin{bmatrix}f(2) & f(1) \\ f(1) & f(0) \end{bmatrix}
$$

最后得到：

$$
\begin{bmatrix}f(n+1) & f(n) \\ f(n) & f(n - 1) \end{bmatrix} = 
\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}^{n}
$$



## 参考代码

```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const ll mod = 1e9 + 7;

struct Matrix
{
    ll m[2][2] = {0};
};

Matrix mul(Matrix a, Matrix b)
{
    Matrix ans;
    for (int i = 0; i < 2; i++)
    {
        for (int j = 0; j < 2; j++)
        {
            for (int k = 0; k < 2; k++)
            {
                ans.m[i][j] = (ans.m[i][j] + a.m[i][k] * b.m[k][j] % mod) % mod;
            }
        }
    }
    return ans;
}

ll fib(ll n)
{
    Matrix res, bsc;
    bsc.m[0][0] = bsc.m[0][1] = bsc.m[1][0] = 1;
    res.m[0][0] = res.m[1][1] = 1;
    while (n)
    {
        if (n & 1)
        {
            res = mul(res, bsc);
        }
        bsc = mul(bsc, bsc);
        n >>= 1;
    }
    return res.m[0][1];
}

int main()
{
    int t;
    scanf("%d", &t);
    while (t--)
    {
        ll n;
        scanf("%lld", &n);
        printf("%lld\n", fib(n) * fib(n + 1) % mod);
    }
    return 0;
}
```



# E. Easy String Problem

## 题意

给出只包含大写字母的字符串 $A$ 和字符串 $B$ ，每次可以交换字符串 $A$ 两个相邻的字符，求 $A$ 变成 $B$ 的最小交换次数。

## 思路

相信大家看到这道题都会感到一丝熟悉，就像给定一个数组，我们每次可以交换数组中相邻的两个元素，求将它变成有序需要的最小的交互次数，也就是**求其逆序对个数**。其实它们的运作方式是完全，只不过这里只是将**有序**的基准变成了**变成** $B$ 。

如果字符串中的字符无法让我们直接完成求解操作，我们需要先利用下标进行一个转换操作。以测试用例为例，我们先对 $A$ 编号，并按字符提取下标：

```
1 2 3 4
C A B C
=>
A: 2
B: 3
C: 1 4
```

接着，我们根据提取的下标对字符串 $B$ 进行编号：

```
C C A B
=>
1 4 2 3
```

最后，我们可以通过对数列 $1,4,2,3$ 求逆序对得到答案啦~

## 参考代码


```cpp
#include <iostream>
#include <cstring>
#include <vector>
using namespace std;
const int maxn = 1e6 + 10;
long long ans, n;
// lt[i] 表示字母 (i+'A') 访问到 st[i] 的下标
// a[] 表示处理之后元素值各不相同的数组，temp[] 是归并过程中用到的中间数组
int lt[26], a[maxn], temp[maxn];
vector<int> st[26]; // st[i][j] 表示第 j 个字母 (i+'A') 所在的下标
string strA, strB;

void mergeSort(int left, int right)
{
    if (left == right)
    {
        return;
    }
    int mid = left + (right - left) / 2;
    int i = left, j = mid + 1, k = left;
    mergeSort(left, mid);
    mergeSort(mid + 1, right);
    while (i <= mid && j <= right)
    {
        if (a[i] <= a[j])
        {
            temp[k++] = a[i++];
        }
        else
        {
            temp[k++] = a[j++];
            ans += mid - i + 1; // 求逆序对
        }
    }
    while (i <= mid)
    {
        temp[k++] = a[i++];
    }
    while (j <= right)
    {
        temp[k++] = a[j++];
    }
    for (int l = left; l <= right; l++)
    {
        a[l] = temp[l];
    }
}

int main()
{
    cin >> n;
    cin >> strA >> strB;
    for (int i = 0; i < n; i++)
    {
        int idx = strA[i] - 'A';
        st[idx].push_back(i + 1);
    }
    for (int i = 0; i < n; i++)
    {
        int idx = strB[i] - 'A';
        a[i + 1] = st[idx][lt[idx]++];
    }
    mergeSort(1, n);
    cout << ans << endl;
    return 0;
}
```


> 求逆序对数有 **归并排序** 和 **树状数组** 两种方式。
这里只给出了归并排序的求解方式，强烈建议去学习一下怎么用树状数组求。



# F. Rize's code

## 思路

签到题。题目原本是求每个矩阵的所有矩阵的元素和之和。但是这题更简单，他要求的是每个元素对最终结果的贡献次数。

一个元素 $A_{x,y}$ 处在哪些子矩阵里面？很明显，只要同时满足 $y$ 处于左右区间之中， $x$ 处于上下区间之中即可。答案是 $x \cdot (n-x+1) \cdot y \cdot (m-y+1)$ 。



# G. A Boring Math Problem

## 思路

该问题需要使用线性筛法， 基于线性筛法， 我们不难得到如下的三个数组， $p,e,p^{e}$ 。它们的含义如下：假设对 $n$ 使用算术基本定理进行分解，有如下形式：

$$
n = p_0 ^ {e_0}  p_1 ^ {e_1} \cdots  p_k ^ {e_k}
$$

那么， $p[n] = p_0,e[n] = e_0,p^e [n] = p_0 ^ {e_0}$ 。

注意到对于正整数 $a$ 和 $b$ ，若 $a$ 和 $b$ 互素， 则 $a^k$ 和 $b^k$ 互素。

又由于 $f(i^k)$ 时积性函数， 所以，对于非素数且非素数次幂的数，令 $g(n) = f(n^k)$ ， 有：

$$
g(n) = g(p^e[n]) \times g( n / p^e[n] )
$$

而对于素数或素数次幂， 有如下公式：
$$
g(p^e) = f(p^{ek}) = 1 + p + p^2 + \cdots + p^{ek} = \frac{p^{ek+1} - 1}{p - 1} 
$$
可在 $O(log ek)$ 的时间内求出答案（快速幂与逆元）。

设 $m$ 为 $n$ 内的素数或素数次幂的个数， 时间复杂度为 $O(n + m log k)$ 。



# H. Image Smoother

## 题意

给出 $n$ 行 $m$ 列矩阵，现要将每个矩阵内的数字用其本身加周围 $8$ 个数字之和的平均数来代表，其中平均数向下取整（若周围不足 $8$ 个数字，则尽可能用完周围的数字）。

## 思路

签到题，直接按题意模拟，注意边界条件即可。

## 参考代码


```c++
#include <iostream>
using namespace std;
int dx[] = {0, 0, 1, -1, 1, -1, -1, 1};
int dy[] = {1, -1, 0, 0, -1, 1, -1, 1};
bool check(int x, int y, int n, int m)
{
    if (x >= 0 && y >= 0 && x < n && y < m)
        return true;
    return false;
}
int num[128][128];
int main()
{
    int n, m;
    cin >> n >> m;
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < m; ++j)
            cin >> num[i][j];
    for (int i = 0; i < n; ++i)
    {
        for (int j = 0; j < m; ++j)
        {
            int cnt = 1;
            int sum = num[i][j];
            for (int k = 0; k < 8; ++k)
            {
                int tx = i + dx[k];
                int ty = j + dy[k];
                if (check(tx, ty, n, m))
                {
                    ++cnt;
                    sum += num[tx][ty];
                }
            }
            cout << (sum / cnt);
            if (j != m - 1)
                cout << " ";
        }
        if (i != n - 1)
            cout << endl;
    }
    return 0;
}
```





# I. Find Spies

## 题意

转换成图的角度理解，给出初始可以访问的节点 $D_i$ 以及访问这些节点需要的代价 $W_i$ ，然后给出一张节点从 $1$ 到 $N$ 编号的**有向图**，求能否通过这些初始节点访问整张图。如果可以则输出 YES 和最小的代价，否则输出 NO 和编号最小的不能访问的节点。

注意：假如通过 $D_1$ 访问到 $D_2$, 并不需要支付 $W_2$ 的代价。

## 思路

其实这是一道 Tarjan 求有向图强连通分量的模板题，大致求解步骤如下：

- 利用所有初始能访问的点跑 Tarjan 去求图的强连通分量（得到强连通分量的时候可以顺便求访问该连通分量最小代价）；
- 通过遍历点判断是否能访问整张图，如果遇到没有访问编号不能则输出 NO 和节点编号；
- 我们将每个强连通分量都视作一个点（术语称缩点），然后统计各个强连通分量的入度；
- 所有入度为 $0$ 的点（强连通分量）访问的最小代价之和，即为我们要的答案。

Q: 为什么是入度为 $0$ 呢？
A: 因为入度非零的点，是可以通过其他点访问，我们没有必要在此花费多余的代价。

## 参考代码


```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 1e4;
const int inf = 0x3f3f3f3f;
struct Edge
{
    int v, next;
} e[maxn];
int head[maxn];

// dfn[i]表示i节点被访问的时间戳
// low[i]表示i及其子节点能回溯到的最小时间戳
int dfn[maxn], low[maxn], Time;

// belong[i] 表示i点所属的连通分量
// sccSize[i] 表示连通分量i的大小，i的取值范围为[0,scc)
// scc 表示连通分量的个数
int belong[maxn], sccSize[maxn], scc;
int Stack[maxn], top; // 数组模拟栈
bool inStack[maxn];   // 点是否在栈中
int n, p, r;

// w[i] 表示通过i点开始访问消耗的代价，如果不能访问，则值为inf
// in[i] 表示连通分量i的入度
// minCost[i] 表示访问连通分量i所需要的最小代价
int w[maxn], in[maxn], minCost[maxn];

void addEdge(int u, int v, int i)
{
    e[i].v = v;
    e[i].next = head[u];
    head[u] = i;
}

// main 函数中调用一次能求出u能访问到的所有连通分量
void Tarjan(int u)
{
    int v;
    dfn[u] = low[u] = ++Time;
    Stack[++top] = u;
    inStack[u] = true;
    for (int i = head[u]; i; i = e[i].next)
    {
        v = e[i].v;
        if (!dfn[v])
        {
            Tarjan(v);
            low[u] = min(low[u], low[v]);
        }
        else if (inStack[v])
        {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (dfn[u] == low[u])
    {
        scc++;
        do
        {
            v = Stack[top--];
            belong[v] = scc;
            sccSize[scc]++;
            inStack[v] = false;
            minCost[scc] = min(minCost[scc], w[v]); // 求能访问该连通分量的最小代价
        } while (u != v);
    }
}

int main()
{
    scanf("%d", &n);
    scanf("%d", &p);
    memset(w, inf, sizeof(w));
    memset(minCost, inf, sizeof(minCost));
    for (int i = 0; i < p; i++)
    {
        int t, tw;
        scanf("%d %d", &t, &tw);
        w[t] = tw;
    }

    scanf("%d", &r);
    for (int i = 1; i <= r; i++)
    {
        int u, v;
        scanf("%d %d", &u, &v);
        addEdge(u, v, i);
    }

    for (int i = 1; i <= n; i++)
    {
        // 该点没有被访问过，且该点初始能通过付出一定代价访问
        if (!dfn[i] && w[i] != inf)
        {
            Tarjan(i);
        }
    }

    for (int i = 1; i <= n; i++)
    {
        // 如果还有点没有访问过，说明不能遍历整张图
        if (!dfn[i])
        {
            printf("NO\n");
            printf("%d", i);
            return 0;
        }
    }
    for (int i = 1; i <= n; i++)
    {
        for (int j = head[i]; j; j = e[j].next)
        {
            int v = e[j].v;
            // 如果i能到v，而且它们属于不同的连通分量，则说明该边给v所属连通分量贡献了一个入度
            if (belong[i] != belong[v])
            {
                in[belong[v]]++; // 统计各连通分量的入度
            }
        }
    }

    int ans = 0;
    for (int i = 1; i <= scc; i++)
    {
        // 如果入度为 0，
        if (!in[i])
        {
            ans += minCost[i];
        }
    }
    printf("YES\n");
    printf("%d", ans);
    return 0;
}
```



> 出题人 Veggie: 虽然题目没能照顾到所有层次的同学，但是希望你们这次有玩得开心 ~



# J. Left 4 Dead 2

## 题意

有 $n$ 个队伍参加求生之路 2 的比赛，并且两两之间会进行一场比赛。求生之路有$4$个精英怪，分别是 Tank, Boommer, Witch, Smoker，他们的血量如题意所示。

$n$ 支队伍按照以下规则排序：

- 按照胜场数从大到小排序。胜场的定义为比赛中杀死精英怪的血量和**严格大于**对手。
- 若胜场数一样，则按照净伤害从大到小排序。净伤害的定义是所有比赛自己杀死的精英怪 $hp$ 和减掉对手杀死的精英怪的 $hp$ 和，不论比赛的胜负。
- 若无法分出胜负，则输出 `No winnner`，否则输出队伍的序号（队伍序号从 $1$ 开始）。

## 思路

签到题，根据题意模拟即可，值得注意的是 $n = 1$ 时，即只有一只队伍时，直接输出 $1$ 。

输入矩阵的解释：矩阵的每一项都有 $4$ 个整数，第 $i$ 行第 $j$ 列的 $4$ 个整数表示的是队伍 $i$ 在和队伍 $j$ 的比赛中杀死的 $4$ 种精英怪的数量，对于矩阵的同一行，每一项用 `|` 隔开

$n=1$ 时，当然也会有输入，即 `0 0 0 0 `

## 参考代码


```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int maxn = 1e2 + 5;

const ll tank_hp = 98000, boomer_hp = 9800, witch_hp = 980, smoker_hp = 98;

struct Contest
{
    ll t, b, w, s;
} contest[maxn][maxn];

struct Team
{
    int idx, wins;
    ll diff;
} team[maxn];

ll get_sum_of_hp(int i, int j)
{
    ll res = contest[i][j].t * tank_hp + contest[i][j].b * boomer_hp + contest[i][j].w * witch_hp + contest[i][j].s * smoker_hp;
    return res;
}

bool cmp(Team a, Team b)
{
    return a.wins == b.wins ? a.diff > b.diff : a.wins > b.wins;
}

int main()
{
    int T;
    scanf("%d", &T);
    while (T--)
    {
        int n;
        scanf("%d", &n);
        char ch[3];
        for (int i = 1; i <= n; i++)
        {
            for (int j = 1; j <= n; j++)
            {
                if (j > 1)
                    scanf("%s", ch);
                scanf("%lld%lld%lld%lld", &contest[i][j].t, &contest[i][j].b, &contest[i][j].w, &contest[i][j].s);
            }
        }
        if (n == 1)
        {
            printf("1\n");
            continue;
        }
        for (int i = 1; i <= n; i++)
        {
            ll diff = 0;
            int wins = 0;
            team[i].idx = i;
            for (int j = 1; j <= n; j++)
            {
                if (i == j)
                    continue;
                ll sum1 = get_sum_of_hp(i, j);
                ll sum2 = get_sum_of_hp(j, i);
                if (sum1 > sum2)
                    wins++;
                diff += sum1 - sum2;
            }
            team[i].diff = diff;
            team[i].wins = wins;
        }
        sort(team + 1, team + n + 1, cmp);
        if (team[1].wins == team[2].wins && team[1].diff == team[2].diff)
            printf("No winner\n");
        else
            printf("%d\n", team[1].idx);
    }
    return 0;
}
```







# K. Cocoa's homework

## 思路

设： $c_1=a_n,c_2=a_{n-1},...,c_n=a_1$

设列向量 $\alpha = [c_1,c_2,...,c_3]^T,\beta = [b_1,b_2,...,b_n]^T$ ，再设 $M'=\alpha\beta^T$ ， $\lambda$ 是 $M'$ 的特征值， $E$ 是单位矩阵。

那么可以知道， $r(M')=1$ 。故线性方程组方程 $M'x=0$ 有 $n-1$ 个线性无关的解，那么 $0$ 是特征多项式方程 $\det(\lambda E-M')=0$ 的至少 $n-1$ 重根。

根据矩阵的迹和特征值的关系，可以知道：

$$
\sum_{i=1}^n c_ib_i=\sum_{i=1}^n \lambda_i
$$

那么剩下的那一个特征值就是 $\sum_{i=1}^n c_ib_i$ 。

可以知道，原答案与 $(-1)^{n(n-1)/2}\det(M'+wE)$ 相同。因为原来的矩阵 $M$ 的每一行需要 $n(n-1)/2$ 相邻行交换才能上下翻转。

又知道矩阵的行列式是特征值的乘积。故：

$$
\det(M'+wE)=(\sum_{i=1}^nc_ib_i+w)w^{n-1}
$$

最终答案为：

$$
\det(M'+wE)=(-1)^{n(n-1)/2}(\sum_{i=1}^nc_ib_i+w)w^{n-1}
$$

修改仅需要简单的维护所有 $a_i,b_i$ 和 $\sum_{i=1}^nc_ib_i$ 的值即可。