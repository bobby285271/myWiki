---
title: Home
description: Life, Love, Linux
published: true
date: 2020-07-31T16:06:52.050Z
tags: 
editor: undefined
---

## 日常搬家

围观了 AOSC 的维基系统之后，觉得拿来当作博客系统也是很合适的。

在搬迁所有博文之后，**会开放注册**。

大家如果认为我的文章可以完善，将可以**自由编辑**，可以获取 Markdown 源码并**规范转载**。

> 禁止恶意编辑，会做防备措施的。

### 公式测试

 这是行内公式 $i+1$，测试。

$$\int_{2}^{1}\frac{f(x)}{g(x)}dx$$

### 代码测试

题目 [Codeforces 1388C](https://codeforces.ml/contest/1388/problem/C)，跑 DFS 每个节点解一遍二元一次方程再判一堆东西就行了，~~赛时差点写不出来~~，菜爆。

测试一下行内代码 `debug(x, ' ', fhappy, ' ', fsad, ' ', endl);` 本地调试会有*高亮*。

```cpp
#if 0
        clear
        g++ -std=c++17 -O2 -Wall -lm "$0" && ./a.out
        exit
#endif
#include <bits/stdc++.h>
#pragma GCC optimize("Ofast")
#pragma GCC target("avx,avx2,fma")
#define _USE_MATH_DEFINES
#define endl '\n'
#define pb emplace_back
#define push_back emplace_back
#define push_front emplace_front
#define push emplace
#define fir first
#define sec second
#define ins insert
#define umap map
#define unordered_map map
#define mset multiset
#define all(x) x.begin(), x.end()
#define rep(i, x, y) for (auto i = (x); i < (y); ++i)
using namespace std;
typedef long long ll;
typedef vector<int> vi;
typedef vector<long long> vll;
typedef pair<int, int> pii;
typedef pair<long long, long long> pll;
#ifdef _USE_DEBUG
clock_t CLOCK_START_TIME = clock();
#endif
const int MAXN = 2e5 + 10;
const int MOD = 1e9 + 7;
const int MOD1 = 1e9 + 9;
const int MOD2 = 998244353;
const int INV2 = 5e8 + 4;
const int INF = 0x3f3f3f3f;
const long long INFLL = 0x3f3f3f3f3f3f3f3f;
long long TESTCASE_NUM = 1;
long long TESTCASE_CUR = 1;
template <typename T>
inline void _read(T &x) { cin >> x; }
inline void _read(int &x) { scanf("%d", &x); }
inline void _read(long long &x) { scanf("%lld", &x); }
inline void _read(float &x) { scanf("%f", &x); }
inline void _read(double &x) { scanf("%lf", &x); }
inline void _read(char &x) { x = getchar(); }
inline void _read(char *x) { scanf("%s", x); }
inline void read() {}
template <typename T, typename... U>
inline void read(T &head, U &... tail)
{
    _read(head);
    read(tail...);
}
template <typename T>
inline void _write(const T &x) { cout << x; }
inline void _write(const int &x) { printf("%d", x); }
inline void _write(const long long &x) { printf("%lld", x); }
inline void _write(const float &x) { printf("%.6f", x); }
inline void _write(const double &x) { printf("%.8lf", x); }
inline void _write(const char &x) { putchar(x); }
inline void _write(const char *x) { printf("%s", x); }
inline void write() {}
template <typename T, typename... U>
inline void write(const T &head, const U &... tail)
{
    _write(head);
    write(tail...);
}
inline void debug() {}
#ifdef _USE_DEBUG
template <typename T, typename... U>
inline void debug(const T &head, const U &... tail)
{
    _write("\033[1m\033[33m\33[4m");
    write(head, tail...);
    _write("\033[0m");
    // system("sleep 1");
    fflush(stdout);
}
#else
template <typename T, typename... U>
inline void debug(const T &head, const U &... tail)
{
}
#endif
void solve();
void init();
template <typename T1, typename T2>
inline T1 max(T1 a, T2 b) { return a < b ? b : a; }
template <typename T1, typename T2>
inline T1 min(T1 a, T2 b) { return a < b ? a : b; }
template <typename T>
inline bool isprime(T p)
{
    if (p == 2 || p == 3)
        return 1;
    if (p < 2 || (p % 6 != 1 && p % 6 != 5))
        return 0;
    for (T i = 5; i * i <= p; i = i + 6)
        if (p % i == 0 || p % (i + 2) == 0)
            return 0;
    return 1;
}
template <typename T1, typename T2, typename T3>
T1 qpow(T1 a, T2 b, T3 m)
{
    T1 ans = 1, base = a;
    while (b)
    {
        if (b & 1)
            ans = ans * base % m;
        base = base * base % m;
        b >>= 1;
    }
    return ans;
}
template <typename T1, typename T2>
T1 qpow(T1 a, T2 b)
{
    T1 ans = 1, base = a;
    while (b)
    {
        if (b & 1)
            ans = ans * base;
        base = base * base;
        b >>= 1;
    }
    return ans;
}
template <typename T1, typename T2>
T1 inv(T1 i, T2 m)
{
    if (i == 1)
        return 1;
    return (m - m / i) * inv(m % i, m) % m;
}

#if 0
#define _USE_UNORDERED_MAP
#undef unordered_map
#undef umap
#define umap unordered_map
#endif

#if 0
#define _USE_PUSH
#undef push_back
#undef push_front
#undef push
#undef pb
#define pb push_back
#endif

#if 0
#define _USE_LONG_LONG
#define int long long
#define pii pair<long long, long long>
#define vi vector<long long>
#define INF 0x3f3f3f3f3f3f3f3f
#endif

void DISPLAY_DEBUG_INFO()
{
#ifdef _USE_DEBUG
    printf("\n\033[1m\033[32m=> \033[1m\033[33m%ld\033[1m\033[32m ms used. ", clock() - CLOCK_START_TIME);
#endif
#if (!defined _USE_PUSH) && (defined _USE_DEBUG)
    printf("\033[1m\033[31mAll \033[1m\033[33mpush\033[1m\033[31m has been replaced by \033[1m\033[33memplace\033[1m\033[31m (Experiment, C++ 11 required).\033[0m ");
#endif
#if (defined _USE_LONG_LONG) && (defined _USE_DEBUG)
    printf("\033[1m\033[32mAll \033[1m\033[33mint\033[1m\033[32m has been replaced by \033[1m\033[33mlong long\033[1m\033[32m.\033[0m ");
#endif
#if (!defined _USE_UNORDERED_MAP) && (defined _USE_DEBUG)
    printf("\033[1m\033[32mAll \033[1m\033[33munordered_map\033[1m\033[32m has been replaced by \033[1m\033[33mmap\033[1m\033[32m.\033[0m ");
#endif
#ifdef _USE_DEBUG
    printf("\033[0m\n\n");
#endif
}

signed main()
{
#ifdef _USE_FILEIO
    freopen("in1.txt", "r", stdin);
    // freopen("ans1.txt", "w", stdout);
#endif
    init();
    scanf("%lld\n", &TESTCASE_NUM);
    while (TESTCASE_NUM--)
    {
        solve();
        TESTCASE_CUR++;
    }
    DISPLAY_DEBUG_INFO();
    return 0;
}

void init() {}

const ll maxn = 1e5 + 10;

ll n, m;
ll p[maxn + 10]; // number of people
ll h[maxn + 10]; // diff: happy-sad
vll g[maxn + 10];
// pll a[maxn + 10]; // fir: happy ,sec: sad
pll err = {-1, -1};

pll dfs(ll x, ll fa)
{
    ll happy = 0, sad = 0;
    for (auto i : g[x])
    {
        if (i == fa)
        {
            continue;
        }
        else
        {
            pll getget = dfs(i, x);
            if (getget == err)
            {
                return err;
            }
            else
            {
                happy += getget.fir;
                sad += getget.sec;
            }
        }
    }
    ll ss = happy + sad + p[x];
    ll fhappy = (ss + h[x]) / 2;
    ll fsad = ss - fhappy;
    if ((ss + h[x]) % 2 != 0)
    {
        return err;
    }
    else if (fhappy < happy || fhappy > ss || fsad > ss)
    {
        return err;
    }
    else
    {
        debug(x, ' ', fhappy, ' ', fsad, ' ', endl);
        return {fhappy, fsad};
    }
}

void solve()
{
    read(n, m);
    for (int i = 1; i <= n; i++)
    {
        g[i].clear();
    }
    for (int i = 1; i <= n; i++)
    {
        read(p[i]);
    }
    for (int i = 1; i <= n; i++)
    {
        read(h[i]);
    }
    for (int i = 1; i < n; i++)
    {
        ll u, v;
        read(u, v);
        g[u].pb(v);
        g[v].pb(u);
    }
    pll getget = dfs(1, 0);
    if (getget == err)
    {
        write("NO", endl);
        return;
    }
    write("YES", endl);
}
```