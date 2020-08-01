---
title: 2020 Nowcoder Basic Algorithm Training Camp Solutions
description: 
published: true
date: 2020-08-01T02:38:35.589Z
tags: algorithm, contest, nowcoder, 2020
editor: markdown
---

颓废了一个寒假，Codeforces 打来打去都是 Specialist，菜的一批。POJ 又刷到怀疑人生。后来被 Kaidora 神犇安排了这个集训营。当时已经是第四场要开始了，还是去了。果然，哪怕都是自闭五小时每小时过一题，氪了金感觉就是不一样。

点击下方的页码查看相应场次的记录，第 $i$ 页对应的是第 $i-1$ 场比赛。

# 2020 牛客寒假算法基础集训营 1
 
> 字符串、贪心、矩阵快速幂、概率论、计算几何、并查集、数论

1A. Honoka 和格点三角形
----------------

### 大意

给出 $m \times n$ 格点矩阵，问在矩阵里面能找出多少个面积为 $1$，且至少有一条边平行于 $x$ 轴或 $y$ 轴的三角形。答案对 $1000000007$ 取模。

### 思路

若存在平行于 $x$ 轴或 $y$ 轴的边且长度为 $2$，一共有 $2(n-1)(m-2)m+2(m-1)(n-2)n$ 种情况。

若存在平行于 $x$ 轴或 $y$ 轴的边且长度为 $1$，除去已经计算过的情况（即还存在平行于$x$ 轴或 $y$ 轴且长度为 $2$ 的边），一共有 $2(m-1)(m-2)(n-2)+2(n-1)(n-2)(m-2)$ 种情况。

相加后化简，得到 $2(m+n-2)(2mn-3m-3n+4)$，利用 $ab\ mod\ c = ((a\ mod\ c)(b\ mod\ c))\ mod\ c$ 计算结果。

    #include <bits/stdc++.h>
    using namespace std;
    int main()
    {
        long long n, m, tempa, tempb;
        cin >> n >> m;
        tempa = 2 * (m + n - 2);
        tempb = (2 * m * n - 3 * m - 3 * n + 4);
        cout << (tempa % 1000000007) * (tempb % 1000000007) % 1000000007 << endl;
        return 0;
    }

1B. Kotori 和 Bangdream
---------------------

### 大意及思路

签到题。求数学期望，也就是可能结果的概率乘以其结果的总和。

    #include <bits/stdc++.h>
    using namespace std;
    int main()
    {
        double n, x, a, b;
        cin >> n >> x >> a >> b;
        double ans = (a * x + b * (100 - x)) * n / 100;
        printf("%.2lf\n", ans);
        return 0;
    }

1C. Umi 和弓道
----------

### 大意

一个人在 $(x_0,y_0)$，给出 $n$ 个靶子，在 $x$ 轴或 $y$ 轴放置挡板，令放置挡板后可以射中的靶子数量不多于 $k$ 个。

### 思路

只要靶子和 $(x_0,y_0)$ 不在一个象限就有可能被挡掉。要想挡掉一个靶子，只需求出靶子和 $(x_0,y_0)$ 所在直线与坐标轴的交点，并保证挡板覆盖了这个点。我们只需要恰好覆盖 $n-k$ 个这样的点就行了。

    #include <bits/stdc++.h>
    using namespace std;
    const double inf = 1e18;
    vector<double> v1, v2;
    int main()
    {
        v1.clear();
        v2.clear();
        double x0, y0;
        int n, k, i;
        cin >> x0 >> y0 >> n >> k;
        k = n - k;
        for (i = 0; i < n; i++)
        {
            double x, y;
            cin >> x >> y;
            if (x * x0 < 0)
            {
                v2.push_back(y0 - x0 * (y - y0) / (x - x0));
            }
            if (y * y0 < 0)
            {
                v1.push_back(x0 - y0 * (x - x0) / (y - y0));
            }
        }
        double mi = inf;
        sort(v1.begin(), v1.end());
        sort(v2.begin(), v2.end());
        if (v1.size() >= k)
        {
            double head = 0, tail = k - 1;
            while (tail < v1.size())
            {
                mi = mi > (v1[tail] - v1[head]) ? (v1[tail] - v1[head]) : mi;
                tail++, head++;
            }
        }
        if (v2.size() >= k)
        {
            double head = 0, tail = k - 1;
            while (tail < v2.size())
            {
                mi = mi > (v2[tail] - v2[head]) ? (v2[tail] - v2[head]) : mi;
                tail++, head++;
            }
        }
        if (mi == 1e18)
            cout << -1;
        else
            printf("%.7lf", mi);
        return 0;
    }

1D. Hanayo 和米饭
-------------

### 大意

给出 $n-1$ 个数，求一个数使得所有的 $n$ 个数经过排序后可形成公差为 $1$ 的等差数列。保证答案存在且唯一。

### 思路

签到题。将 $n-1$ 个数排序，看相邻两数的差是否为 $2$。

    #include <bits/stdc++.h>
    using namespace std;
    int main()
    {
        int n;
        cin >> n;
        int a[n - 1];
        for (int i = 0; i < n - 1; i++)
        {
            cin >> a[i];
        }
        sort(a, a + n - 1);
        for (int i = 0; i < n - 2; i++)
        {
            if (a[i + 1] - a[i] != 1)
            {
                cout << a[i] + 1 << endl;
                break;
            }
        }
        return 0;
    }

1E. Rin 和快速迭代
------------

### 大意

令 $f(x)$ 为 $x$ 因子个数，将 $f$ 迭代下去，问迭代多少次能得到 $2$。

### 思路

模拟就行。求因子个数时注意处理完全平方数。

    #include <bits/stdc++.h>
    using namespace std;
    long long sol(long long x)
    {
        long long temp = 0;
        long long i;
        for (i = 1; i * i <= x; i++)
        {
            if (x % i == 0)
                temp++;
        }
        temp = temp << 1;
        i--;
        if (i * i == x)
            temp--;
        return temp;
    }
    int main()
    {
        long long n;
        cin >> n;
        int ans = 0;
        while (1)
        {
            n = sol(n);
            ans++;
            if (n == 2)
            {
                cout << ans << endl;
                return 0;
            }
        }
        return 0;
    }

1F. Maki 和 Tree
--------------

### 大意

在树（无向）中有黑白两种点。求 $n$ 个点的树中有多少简单路径有且仅有通过一个黑色点。

### 思路

先预处理各个白色连通及其大小，具体思路是读入一条边如果这条边的两端都是白色连通块就连在一起（并查集）。然后再算各个黑点的答案。

    #include <bits/stdc++.h>
    using namespace std;
    #define ll long long
    int fa[111111];  //父亲
    int kdm[111111]; //孩子数量
    string color;
    int f(int x)
    { //寻找祖先
        if (fa[x] == x)
            return x;
        return f(fa[x]);
    }
    void uni(int x, int y)
    { //连接 x 点和 y 点
        int p = f(x), q = f(y);
        if (p != q)
        {
            if (kdm[p] > kdm[q])
            {
                fa[q] = p;
                kdm[p] += kdm[q] + 1;
            }
            else
            {
                fa[p] = q;
                kdm[q] += kdm[p] + 1;
            }
        }
    }
    ll t[111111]; //统计连通块白点数量
    vector<int> g[111111];
    ll gao(vector<int> temp)
    { //temp 为黑点的每个相邻白点孩子数量集合
        ll res = 0, i;
        ll n = temp.size();
        if (n == 0)
            return 0;
        ll dp[n] = {0}, sum[n] = {0}, s = 0;
        sum[0] = s = temp[0];
        for (i = 0; i < n; i++)
        {
            res += temp[i];
        }
        for (i = 1; i < n; i++)
        {
            sum[i] = s += temp[i];
        }
        for (i = 1; i < n; i++)
        {
            dp[i] = dp[i - 1] + temp[i] * sum[i - 1];
        }
        return res + dp[n - 1];
    }
    int main()
    {
    
        int n, i, j;
        cin >> n >> color;
        for (i = 1; i <= n; i++)
            fa[i] = i;
        for (i = 1; i < n; i++)
        {
            int x, y;
            cin >> x >> y;
            g[x].push_back(y);
            g[y].push_back(x);
            if (color[x - 1] == 'W' && color[y - 1] == 'W')
                uni(x, y);
        }
        ll sum = 0;
        for (i = 1; i <= n; i++)
            t[i] = kdm[f(i)] + 1;
        for (i = 1; i <= n; i++)
        {
            if (color[i - 1] == 'B')
            {
                vector<int> temp;
                for (j = 0; j < g[i].size(); j++)
                {
                    if (color[g[i][j] - 1] == 'W')
                        temp.push_back(t[g[i][j]]); //若相邻点是白点，加入 temp
                }
                sum += gao(temp);
            }
        }
        cout << sum;
    }

1G. Eli 和字符串
-----------

### 大意

给出一个字符串，求连续子串的最小长度，要求这个连续子串要包含至少 $k$ 个相同的某个字母。

### 思路

前缀和加尺取法。前缀和用于求某个连续子串各个字母出现的次数。尺取法即连续子串两个指针，右指针不断右移，当发现满足条件的连续子串则左指针开始右移，当右指针移动到尽头则停止。

    #include <bits/stdc++.h>
    using namespace std;
    int cnt[26][200010];
    int main()
    {
        int n, k;
        string a;
        cin >> n >> k >> a;
        int l = a.size();
        for (int i = 0; i < l; i++)
        {
            for (int j = 0; j < 26; j++)
            {
                cnt[j][i + 1] = cnt[j][i];
            }
            cnt[a[i] - 'a'][i + 1]++;
        }
        int p1 = 0, p2 = 1, ans = n + 1;
        while (1)
        {
            int flag = 0;
            for (int i = 0; i < 26; i++)
            {
                if (cnt[i][p2] - cnt[i][p1] >= k)
                {
                    ans = min(ans, p2 - p1);
                    flag = 1;
                }
            }
            if (flag == 1)
                p1++;
            else
                p2++;
            if (p2 == n + 1)
                break;
        }
        if (ans == n + 1)
            cout << "-1" << endl;
        else
            cout << ans << endl;
        return 0;
    }

1H. Nozomi 和字符串
--------------

### 大意

一个 $01$ 串，允许你改变其中的 $k$ 个数字，然后找出一个尽可能长的连续子串，这个子串上的所有字符都相同。求这个子串的长度。

### 思路

依然是尺取法。分成将 $0$ 变成 $1$ 和将 $1$ 变成 $0$ 两种情况。对于前者，还是两个指针，右指针不断右移，当遇到 $0$ 就用掉一次操作直到次数用尽。接下来右指针继续右移直到再次需要操作的时候，这时左指针右移直到已用操作数恰好减一为止，这时右指针继续右移。后者同理。

    #include <bits/stdc++.h>
    using namespace std;
    int main()
    {
        string s;
        int n, k, ans = 0;
        cin >> n >> k >> s;
        int pl = 0, pr = 0, change = 0;
        // 0 => 1
        for (int i = 0; i < n; i++)
        {
            if (s[i] == '0')
            {
                if (change < k)
                {
                    change++;
                }
                else
                {
                    while (pl <= pr && s[pl] != '0')
                        pl++;
                    pl++;
                }
            }
            pr++;
            ans = max(ans, pr - pl);
        }
        pl = 0, pr = 0, change = 0;
        for (int i = 0; i < n; i++)
        {
            if (s[i] == '1')
            {
                if (change < k)
                {
                    change++;
                }
                else
                {
                    while (pl <= pr && s[pl] != '1')
                        pl++;
                    pl++;
                }
            }
            pr++;
            // cout << pl << " " << pr << endl;
            ans = max(ans, pr - pl);
            // cout << ans << endl;
        }
        cout << ans << endl;
        return 0;
    }

1I. Nico 和 Niconiconi
--------------------

### 大意

给出一个字符串，其中 `nico` 计 $a$ 分，`niconi` 计 $b$ 分，`niconiconi` 计 $c$ 分，每个字符都只能参与一次计分，问最大分数。

### 思路

简单 DP。$dp[i]$ 可由 $dp[i-1]$、$dp[i-3]$、$dp[i-5]$ 和 $dp[i-9]$ 转移而来，转移后取最大者即可。要注意避免越界。

    #include <bits/stdc++.h>
    using namespace std;
    long long dp[300010];
    int main()
    {
        string s;
        long long n, a, b, c;
        cin >> n >> a >> b >> c >> s;
        for (long long i = 0; i < n; i++)
        {
            if (i > 0)
                dp[i] = dp[i - 1];
            if (i >= 3 && s.substr(i - 3, 4) == "nico")
                dp[i] = max(dp[i], dp[i - 3] + a);
            if (i >= 5 && s.substr(i - 5, 6) == "niconi")
                dp[i] = max(dp[i], dp[i - 5] + b);
            if (i >= 9 && s.substr(i - 9, 10) == "niconiconi")
                dp[i] = max(dp[i], dp[i - 9] + c);
        }
        cout << dp[n - 1] << endl;
        return 0;
    }

1J. ## TODO##
------------

# 2020 牛客寒假算法基础集训营 2

> 枚举、贪心、DP、数论、思维、数据结构、哈希

2A. 做游戏
------

### 大意

石头剪刀布。已知两人分别出了多少次石头、剪刀和布，求其中一个玩家最多能获胜多少局。

### 思路

贪心。让尽量多的剪刀 \- 布、石头 \- 剪刀和布 \- 石头成组，可以直接出答案 $min(A,Y)+min(B,Z)+min(C,X)$。

    #include <bits/stdc++.h>
    using namespace std;
    int main()
    {
        long long a, b, c, x, y, z;
        cin >> a >> b >> c >> x >> y >> z;
        cout << min(a, y) + min(b, z) + min(c, x) << endl;
        return 0;
    }

2B. 排数字
------

### 大意

给出一个字符串 $S$，问将字符串打乱后最多能有多少个不同的连续子串为 $616$ 。 

### 思路

只需将所有的 $6$ 和 $1$ 集中起来：$6161616...$ ，因此统计 $6$ 和 $1$ 的个数即可，得出答案。注意 $cnt6$ 个 $6$ 最多能只造 $cnt6-1$ 个满足题意的字串，所以是 $min(cnt6-1,cnt1)$。如果算出来小于零应该将答案调整为 $0$。

    #include <bits/stdc++.h>
    using namespace std;
    int main()
    {
        int n, cnt1 = 0, cnt6 = 0;
        string s;
        cin >> n >> s;
        for (int i = 0; i < n; i++)
        {
            if (s[i] == '1')
                cnt1++;
            else if (s[i] == '6')
                cnt6++;
        }
        cout << max(0, min(cnt6 - 1, cnt1)) << endl;
        return 0;
    }

2C. 算概率
------

### 大意

一共 $n$ 道题，第 $i$ 道题的正确率是 $p_i$​，给出这个正确率对 $10^9+7$ 取模的结果，求这 $n$ 题里恰好有 $0,1,\cdots,n$ 题正确的概率分别是多少，同样对 $10^9+7$ 取模。对分数取模的定义已在题目给出。

### 思路

简单 DP。一个难点是求状态转移方程，一个是理解题目中对分数取模的概念（本蒟蒻在补第四套题时才会的乘法逆元，做这道题时直接观察样例对 $\frac{1}{2}$ 取模的结果，然后分子分母代进 $a$ 和 $b$ 蒙一把，发现简单粗暴地将概率乘个 $(10^9+7)+1$ 就是了）。要求前 $i$ 题正确 $j$ 题的概率，首先要知道的是前面具体过了哪几道题我们不需要知道（这里有点靠直觉），我们只需要简单粗暴地把它当作一个整体参与后面的计算。接下来，我们要分两种情况讨论，一个是第 $i$ 题对了 $f_{i-1,j} \cdot p_i$，一个是第 $i$ 题错了 $f_{i,j} \cdot (1-p_i)$，加起来就是了。状态转移方程 $f_{i,j} = f_{i-1,j} \cdot p_i+f_{i-1,j} \cdot (1-p_i)$。

    #include <bits/stdc++.h>
    using namespace std;
    
    long long n, p[2005], dp[2005][2005];
    
    int main()
    {
        cin >> n;
        for (int i = 1; i <= n; i++)
        {
            cin >> p[i];
        }
        dp[0][0] = 1;
        for (int i = 1; i <= n; i++)
        {
            dp[i][0] = dp[i - 1][0] * (1000000008 - p[i]) % 1000000007;
            for (int j = 1; j <= i; j++)
                dp[i][j] = (dp[i - 1][j] * (1000000008 - p[i]) + dp[i - 1][j - 1] * p[i]) % 1000000007;
        }
        for (int i = 0; i <= n; i++)
            cout << dp[n][i] << ' ';
        return 0;
    }
    

2D. 数三角
------

### 大意

给出 $n$ 个点，问这 $n$ 个点形成的三角形中，总共有多少个钝角三角形。

### 思路

签到。枚举后比较 $a^2+b^2$ 和 $c^2$ 即可，另外注意三点不能共线。

    #include <bits/stdc++.h>
    using namespace std;
    
    int n, a[505], b[505], ans = 0;
    
    bool f(int i, int j, int k)
    {
        int check1 = (a[j] - a[i]) * (a[k] - a[i]) + (b[j] - b[i]) * (b[k] - b[i]);
        int check2 = (a[j] - a[i]) * (b[k] - b[i]) - (a[k] - a[i]) * (b[j] - b[i]);
        return check1 < 0 && check2 != 0;
    }
    int main()
    {
        cin >> n;
        for (int i = 0; i < n; i++)
            cin >> a[i] >> b[i];
        for (int i = 0; i < n; i++)
        {
            for (int j = i + 1; j < n; j++)
            {
                for (int k = j + 1; k < n; k++)
                {
                    if (f(i, j, k) || f(k, i, j) || f(j, i, k))
                    {
                        ans++;
                    }
                }
            }
        }
    
        cout << ans << endl;
        return 0;
    }

2E. 做计数
------

### 大意

求有多少个不同的正整数三元组 $(i,j,k)$ 满足 $\sqrt i+\sqrt j = \sqrt k$，而且 $i \cdot j \leq n$。

### 思路

看过一样的题，所以说多去看百度贴吧的营销号推送确实能涨知识。见到根号疯狂平方就对了（必要时候还是要移项的）。等式左右两边平方，$i+j+2 \sqrt {ij}=k$，发现要使 $k$ 为整数，必需要令 $i \cdot j$ 为完全平方数，所以我们就枚举完全平方数，然后再枚举这些完全平方数的因数。

    #include <bits/stdc++.h>
    using namespace std;
    int n, ans;
    int main()
    {
        cin >> n;
        for (int i = 1; i * i <= n; i++)
        {
            for (int j = 1; j <= i; j++)
                if (i * i % j == 0)
                    ans += 2;
            ans--;
        }
        cout << ans << endl;
        return 0;
    }

2F. 拿物品
------

### 大意

多个物品，每个物品两个属性 $a$ 和 $b$， 甲乙两人轮流拿物品，最后甲的得分为其拿到物品的 $a$ 属性的和，乙则为 $b$ 属性的和。问两人最优策略下会拿哪些物品。

### 思路

假设两人赛后互换物品 $(a_1,b_1)$ 和 $(a_2,b_2)$，那么两人得分变化分别为 $-a_1+a_2$ 和 $b_1-b_2$，如果想让前者得分更大，要有 $a_1+b_1 < a_2+b_2$。后者同理。所以就按照 $a+b$ 排序。

    #include <bits/stdc++.h>
    using namespace std;
    
    const int N = 2e5 + 7;
    int n;
    vector<int> sa, sb;
    struct temp
    {
        int a, b, id;
    } a[N];
    
    bool cmp(temp a, temp b)
    {
        return a.a + a.b > b.a + b.b;
    }
    
    int main()
    {
        cin >> n;
        for (int i = 1; i <= n; i++)
            cin >> a[i].a;
        for (int i = 1; i <= n; i++)
        {
            cin >> a[i].b;
            a[i].id = i;
        }
        sort(a + 1, a + 1 + n, cmp);
        for (int i = 1; i <= n; ++i)
            ((i & 1) ? sa : sb).push_back(a[i].id);
        for (auto i : sa)
            cout << i << " ";
        cout << endl;
        for (auto i : sb)
            cout << i << " ";
        cout << endl;
        return 0;
    }

2G. 判正误
------

### 大意

给出七个整数 $a, b, c, d, e, f, g$ 问是否有 $a^d+b^e+c^f=g$。数据范围 $10^9$ 级别，数据量 $10^3$ 级别。

### 思路

在模的意义下上快速幂验证，也就是验证 $a^d+b^e+c^f \equiv g \left ( mod \ M\right )$。模数越多，数据越水，就越稳。

    #include <bits/stdc++.h>
    using namespace std;
    const int mod[] = {2, 3, 5, 7, 11, 31, 71, 97, 233, 397, 433, 449, 607, 857, 10007, 21179, 36251, 44579, 62003, 72883, 97843, 139991, 232013, 369353, 681521, 692711, 777241, 822821, 1956761, 2145137, 2915837, 6229117, 7788787, 13743493, 17331841, 19260817, 19269293, 19959809, 21006959, 23937083, 24410849, 28452757, 28478603, 29229359, 35570827, 35604011, 35875487, 37370863, 38303347, 38475517, 38819149, 40455791, 44021539, 45641993, 46531301, 48866749, 50529641, 52634191, 52790587, 55180799, 56971613, 58259351, 60954737, 62207269, 63367453, 65072599, 66017821, 67952779, 69475349, 74689217, 77059907, 77907121, 79391659, 84768797, 85584601, 85724879, 85756609, 86850899, 91783511, 92331541, 94519499, 96375241, 99033413, 99486311, 100569829, 106873549, 109329881, 109913681, 111186487, 111894067, 112136617, 112417363, 114011921, 119143363, 122994493, 123747781, 124001021, 126515639, 128191039, 128767909, 132222763, 133587661, 139644719, 145641527, 153388423, 155187077, 156883333, 157989581, 159538063, 161488643, 164039129, 166070447, 169181543, 169554227, 173564801, 175742867, 185469637, 187203899, 191263223, 198691817, 204144887, 211631201, 217903877, 218028203, 220073423, 228143453, 228667423, 232064653, 240519263, 245647159, 247586411, 247936121, 250949197, 253413211, 253464329, 260572729, 260590409, 262887773, 265711423, 266763641, 273585149, 276472817, 276500531, 280543667, 280649591, 281385491, 291366337, 293273159, 296973107, 302890501, 306568693, 315614297, 316729409, 317617121, 320337781, 320613497, 321322823, 324691051, 325963067, 327184157, 329900633, 330670159, 332058781, 332213669, 332300869, 334382221, 341895677, 347938237, 349011827, 349347503, 349906439, 353796941, 364557253, 364755931, 367946441, 372413831, 374358983, 379589897, 381149689, 389431873, 404683493, 405216109, 405495029, 408142403, 408989747, 410841979, 410935093, 412405351, 412592459, 412722139, 412990573, 418171483, 421270357, 424233613, 427938449, 428492083, 429962881, 430883569, 434988383, 435941201, 438816151, 440052953, 440143589, 444693631, 453646433, 455847109, 456640189, 457911511, 458185237, 463116761, 463861417, 469275953, 471298573, 471712513, 478267417, 483824813, 494828483, 497397293, 499657393, 507957479, 512906621, 519346459, 519879973, 520094713, 523213693, 525673273, 529575763, 529883803, 533887031, 534260809, 535328309, 541992667, 542253071, 544780177, 545567609, 552922529, 555129893, 555820037, 558473471, 563484017, 571310471, 578121241, 582251063, 583825639, 584121323, 592038487, 599098811, 601467677, 610073969, 615059213, 619220713, 622457177, 627412609, 630547919, 632342989, 637357363, 638865419, 648268013, 650007487, 651564761, 654115433, 661281713, 662664461, 667914281, 682988213, 691099121, 691445809, 692038043, 692411953, 698620943, 699007259, 701164631, 706806461, 707096251, 707697451, 709566589, 719095829, 725756807, 736880491, 739603867, 743026709, 744236861, 744396049, 747393791, 749395103, 760341121, 762934307, 773124059, 773195911, 776162609, 781629113, 781884613, 786120631, 788314343, 788898377, 788939293, 790209983, 791933183, 796328783, 798643889, 802280047, 803293991, 803847559, 809752739, 818520473, 820434047, 826810489, 829359959, 829707427, 836587463, 841011167, 843763253, 849410557, 851226437, 853058471, 853168793, 853778327, 859086391, 860720017, 863193077, 873061181, 888803059, 893035529, 900902953, 904636883, 917949577, 921817139, 922328707, 931449133, 933074827, 933156233, 935241721, 935632799, 939948881, 957119773, 961329913, 965269573, 965337949, 967551691, 971080093, 973578143, 976825877, 985100197, 985413691, 986124823, 990650057, 998244353, 999058883, 1000000007};
    int ff(int a, int m)
    {
        return (a % m + m) % m;
    }
    int Pow(int a, int b, int c)
    {
        int ret = 1;
        while (b)
        {
            if (b & 1)
                ret = ret * 1ll * a % c;
            a = a * 1ll * a % c;
            b >>= 1;
        }
        return ret;
    }
    int a, b, c, d, e, f, g;
    
    bool check(int m)
    {
        return ((Pow(ff(a, m), d, m) + Pow(ff(b, m), e, m)) % m + Pow(ff(c, m), f, m)) % m == ff(g, m);
    }
    
    int main()
    {
        int T;
        cin >> T;
        while (T--)
        {
            cin >> a >> b >> c >> d >> e >> f >> g;
            bool flag = 1;
            for (int i = 0; i < 349; ++i)
            {
                if (!check(mod[i]))
                {
                    flag = 0;
                    break;
                }
            }
            puts(flag ? "Yes" : "No");
        }
        return 0;
    }

第三场。

# 2020 牛客寒假算法基础集训营 4

> 搜索、简单 STL、前缀和、二分搜索、位运算、贪心、分治、树

4A. 欧几里得
-------

### 大意

给出了一个递归实现的 GCD 的代码，告诉你递归次数，求最开始的两个数，它们不相同且都是非负数，使这两个数的和最小。

### 思路

签到题。递归次数为 $0$ 次的时候肯定这两个是 $1$ 和 $0$。递归次数为 $1$ 次的时候则是 $2$ 和 $1$。假设说 $a$ 大于 $b$，其实就是已知 $b$ 和 $a\ mod\ b$，然后要让 $a$ 最小，那么就让 $\left \lfloor \frac{a}{b} \right \rfloor$ 最小。那就让它等于 $1$，可以很快发现是斐波那契数列。

    #include <bits/stdc++.h>
    using namespace std;
     
    int main()
    {
        int T;
        cin >> T;
        while (T--)
        {
            long long n;
            cin >> n;
            long long a[n + 10];
            a[0] = 1;
            a[1] = 2;
            for (int i = 2; i < n + 7; i++)
            {
                a[i] = a[i - 1] + a[i - 2];
            }
            if (n == 0)
                cout << "1" << endl;
            else
                cout << a[n + 1] << endl;
        }
        return 0;
    }

4B. 括号序列
-------

### 大意

给出一个仅包含 `[`、`]`、`(`、`)`、`{`、`}` 六种字符的括号序列，判断其是否合法。

### 思路

签到题。开一个栈来储存左括号，读到右括号看看栈顶的左括号和它匹不匹配。匹配就 POP 一个左括号。每时每刻都判断一下栈是不是空的。

    #include <bits/stdc++.h>
    using namespace std;
     
    int main()
    {
        string a;
        cin >> a;
        stack<char> b;
        for (int i = 0; i < a.size(); i++)
        {
            // cout << a[i];
            if (a[i] == '[')
                b.push('[');
            else if (a[i] == '{')
                b.push('{');
            else if (a[i] == '(')
                b.push('(');
     
     
            else if (a[i] == ']')
            {
                if (b.empty() || (! b.empty() && b.top() != '['))
                {
                    cout << "No" << endl;
                    return 0;
                }
                else if (! b.empty())
                    b.pop();
            }
            else if (a[i] == ')')
            {
                if (b.empty() || (! b.empty() && b.top() != '('))
                {
                    cout << "No" << endl;
                    return 0;
                }
                else if (! b.empty())
                    b.pop();
            }
            else if (a[i] == '}')
            {
                if (b.empty() || (! b.empty() && b.top() != '{'))
                {
                    cout << "No" << endl;
                    return 0;
                }
                else if (! b.empty())
                    b.pop();
            }
        }
        if (b.empty())
            cout << "Yes" << endl;
        else
            cout << "No" << endl;
        return 0;
    }

4C. 子段乘积
-------

### 大意

给出一个数列和一个数字 $k$，求其长度为 $k$ 的连续子段的乘积对 $998244353$ 取模余数的最大值。

### 思路

其实会乘法逆元的话就可以尺取了，可惜我刚开始做这题的时候不会，只好线段树了。反正期间也不用改动数字，懒标记什么的统统不要，把线段树建起来直接开始查询。

    #include <bits/stdc++.h>
     
    using namespace std;
     
    const int maxn = 200010;
     
    int a[maxn + 2];
     
    struct tree
    {
        int l, r;
        long long pre, add;
    } t[4 * maxn + 2];
     
    void bulid(int p, int l, int r)
    {
        t[p].l = l;
        t[p].r = r;
        if (l == r)
        {
            t[p].pre = a[l];
            return;
        }
        int mid = l + r >> 1;
        bulid(p * 2, l, mid);
        bulid(p * 2 + 1, mid + 1, r);
        t[p].pre = (t[p * 2].pre * t[p * 2 + 1].pre) % 998244353;
    }
     
    long long ask(int p, int x, int y)
    {
        if (x <= t[p].l && y >= t[p].r)
            return t[p].pre;
        int mid = t[p].l + t[p].r >> 1;
        long long ans = 1;
        if (x <= mid)
            ans = ans * ask(p * 2, x, y) % 998244353;
        if (y > mid)
            ans = ans * ask(p * 2 + 1, x, y) % 998244353;
        return ans;
    }
     
    int main()
    {
        int n, m;
        scanf("%d%d", &n, &m);
        for (int i = 1; i <= n; i++)
            scanf("%d", &a[i]);
        bulid(1, 1, n);
        long long ans = -1;
        for (int i = 1; i + m - 1 <= n; i++)
        {
            int x, y;
            ans = max(ans, ask(1, i, i + m - 1));
        }
        cout << ans << endl;
        return 0;
    }

4D. 子段异或
-------

### 大意

给出一个数列，让你需要输出其中异或值为 $0$ 的不同连续子段的数量。

### 思路

首先用到了前缀和的思想，将前缀异或存进一个数组里。然后接合异或的性质：如果 $a\ xor\ b=c$，那么 $c\ xor\ a=b$，那么数组里任意两个数异或就是某一个区间的异或值，反正我们也不用关心这个区间从哪开始从哪结束，又因为 $a\ xor\ a=0$，我们给数组排下序，看看相邻两个数字是否相等就可以啦。

    #include <bits/stdc++.h>
    using namespace std;
     
    int main()
    {
        int n;
        cin >> n;
        long long a[n + 10];
        long long temp;
        cin >> temp;
        a[0] = temp;
        for (int i = 1; i < n; i++)
        {
            cin >> temp;
            a[i] = a[i - 1] ^ temp;
        }
        sort(a, a + n);
        long long ans = 0, cnt = 1;
        for (int i = 0; i < n; i++)
        {
            if (a[i] == 0)
                ans++;
            if (i != 0 && a[i] == a[i - 1])
                cnt++;
            if (i != 0 && (a[i] != a[i - 1] || i == n - 1))
            {
                ans += cnt * (cnt - 1) / 2;
                cnt = 1;
            }
        }
        cout << ans << endl;
        return 0;
    }

4E. 最小表达式
--------

### 大意

给出一个字符串，里面只包含 $1$ 到 $9$ 还有加号。要求给出字符串的一个排列，使排列后是一个合法的算式而且算式的计算结果最小。

### 思路

贪心。加号的数字知道后就知道要有多少个数相加。然后我们将大的数字放在个位十位这种低位数，将小的数字放在高位数。然后是大数加法。

    #include <bits/stdc++.h>
    using namespace std;
    
    int cnt[20];
    int sum[500050];
    
    string s;
    int main()
    {
        cin >> s;
        int ccnt = 1;
        int n = s.size();
        for (int i = 0; i < n; i++)
        {
            if (isdigit(s[i]))
            {
                cnt[s[i] - '0']++;
            }
            else
            {
                ccnt++;
            }
        }
        int p = 0, cp = 0;
        for (int i = 10; i >= 1; i--)
        {
            while (cnt[i])
            {
                sum[cp] += i;
                cnt[i]--;
                p = (p + 1) % ccnt;
                if (p == 0)
                    cp++;
            }
        }
        for (int i = 0; i < 500010; i++)
        {
            sum[i + 1] += sum[i] / 10;
            sum[i] %= 10;
        }
        int opt = 0;
        for (int i = 500010; i >= 0; i--)
        {
            if (opt || sum[i])
            {
                cout << sum[i];
                opt = 1;
            }
        }
        cout << endl;
        return 0;
    }

4J. 二维跑步
-------

### 完整题目

一个点在平面直角坐标系中移动，初始位置 $(0,0)$，移动了 $n$ 次。从 $(i,0)$ 可以移动到 $(i+1,0)$、$(i+1,1)$、$(i+1,2)$、$(i,0)$、$(i-1,1)$、$(i-1,2)$，从 $(i,1)$ 可以移动到 $(i+1,0)$、$(i+1,1)$、$(i+1,2)$、$(i,1)$、$(i-1,0)$、$(i-1,2)$，从 $(i,2)$ 可以移动到 $(i+1,0)$、$(i+1,1)$、$(i+1,2)$、$(i,2)$、$(i-1,0)$、$(i-1,1)$。这里的 $(i,0)$ 移动到 $(i,0)$ 没有打错，而是不变换坐标的前提下消耗一个步数。已知 $n$ 和 $m$，数值均小于 $3 \cdot 10^6$，然后问你有多少种方式使得点的 $x$ 坐标最后落在 $[-m,m]$，答案对 $998244353$ 取模输出。

### 思路

#### 坐标等价

理解题目在讲啥后，首先要做的是简化题目条件，你会发现这么多的坐标其中纵坐标只有三种，$0$、$1$ 和 $2$，当横坐标增加的时候（也就是从 $i$ 变成 $i+1$ 了，后面类似），无论初始位置的纵坐标是啥，能够到达的位置的纵坐标都是任意的。当横座标不变，纵坐标都是不变的。当横坐标减少的时候，无论初始位置的纵坐标是啥，能够到达的位置的纵坐标都只有两个可选而且都不等于初始位置的纵坐标。综上所述，我们发现纵坐标在这题是无关紧要的，横坐标相同的三个点是等价的。那么我们就可以将题目理解为：从 $x=i$ 的点到 $x=i+1$ 的点有三种方法，到 $x=i$ 的点有一种方法，到 $x=i-1$ 的点有两种方法。

#### 排列组合

考虑到 $n$ 步下来本质还是左移、右移、不动三种方式的组合，只需设出三种方式分别的步数，用组合数公式即可。这里不妨设 $x$ 坐标不变的次数为 $i$，其中 $a$ 次 $x$ 坐标增加了，那么就会有 $n-i-a$ 次坐标减少。我们可以知道最后的坐标位置为 $x_{final}=a-(n-i-a)=2a-n+i$。为了让这个坐标处于 $[-m,m]$ 这个区间，我们要有：

$\left\{\begin{matrix} -m \leq 2a-n+i \leq m \ 0 \leq a \leq n-i \end{matrix}\right. $

由于 $a$ 还是一个整数，算出来的结果还需要上下取整：

$max(\left \lceil \frac{-m+n-i}{2} \right \rceil,0)\leq a \leq min(\left \lfloor \frac{m+n-i}{2} \right \rfloor,n-i)$

结合高中数学内容，我们知道 $i$ 次坐标不变，$a$ 次坐标增加的方案数一共是 $C _{n}^{i}(C _{n-i}^{a} \cdot 3^{a} \cdot 2^{n-i-a})$，我们进行求和操作，得出的结果是：

$ans = \sum_{i=0}^{n} \textrm{C} _{n}^{i}(\sum_{a=max(\left \lceil \frac{-m+n-i}{2} \right \rceil,0)}^{min(\left \lfloor \frac{m+n-i}{2} \right \rfloor,n-i)} \textrm{C} _{n-i}^{a} \cdot 3^{a} \cdot 2^{n-i-a}) $

#### 优化

考虑到 $n$ 和 $m$ 的数据范围，直接进行计算肯定是不行，我们尝试将上面的式子拆为多部分。令：

$\left\{\begin{matrix}f(p,q)=C_p^q \cdot3^q \cdot 2^{p-q} \ L(i)=max(\left \lceil \frac{-m+n-i}{2} \right \rceil,0) \ R(i)=min(\left \lfloor \frac{m+n-i}{2} \right \rfloor,n-i) \ G(i)=\sum_{a=L(i)}^{R(i)}f(n-i,a) \ ans = \sum_{i=0}^{n}C_{n}^{i} G(i) \end{matrix}\right.$

在这些式子中，我们发现要缩短 $ans$ 的计算时间，就必须缩短 $G(i)$ 的计算时间。考虑到计算 $G(i)$ 时 $L(i)$ 和 $R(i)$ 都是一次性计算完成，但是 $f(p,q)$ 这样的式子我们要算上很多遍，我们我们就尝试优化 $f(p,q)$ 的计算。所谓优化很多时候就是预处理。我们看看对应的式子：组合数，得算阶乘吧，得算 $2$ 的 $p-q$ 次幂吧，得算 $3$ 的 $q$ 次幂吧，全部预处理掉。接下来就比较玄学了，考虑到我们除了要计算 $f(n-i,a)$，还要计算 $f(n-i-1,a)$、$f(n-i+1,a)$ 等等，因为恰好有 $C_p^q=C_{p-1}^{q}+C_{p-1}^{q-1}$ 这个公式，也就有了前项推后项的思路。我们尝试将 $2$ 和 $3$ 的指数和组合数匹配一下，得出 $f(p,q)=2f(p-1,q)+3f(p-1,q-1)$ 这样的式子。

接下来就不用再管 $f(p,q)$ 等于什么了，回到 $G(i)$ 这个层面。我们用上面的结论尝试展开上面的式子：

$G(i)=\sum_{a=L(i)}^{R(i)}f(n-i,a)$  
$=\sum_{a=L(i)}^{R(i)}(3f(n-(i+1),a-1)+2f(n-(i+1),a))$  
$=3f(n-(i+1),L(i)-1)+2f(n-(i+1),L(i))+3f(n-(i+1),L(i))+ \cdots +2f(n-(i+1),R(i)-1))+3f(n-(i+1),R(i)-1)+2f(n-(i+1),R(i))$  
$=5 \sum_{a=L(i)}^{R(i)-1}f(n-(i+1),a)+3f(n-(i+1),L(i)-1)+2f(n-(i+1),R(i))$

你会发现有一项是 $\sum_{a=L(i)}^{R(i)-1}f(n-(i+1),a)$，和 $G(i+1)$ 的形式很像，但是后者是 $\sum_{a=L(i+1)}^{R(i+1)}f(n-(i+1),a)$，还是有点区别。怎么办呢？就人为创造一个 $\sum_{a=L(i+1)}^{R(i+1)}f(n-(i+1),a)$ 呗。还是上面那个式子，我们把所有的 $L(i)$ 换成 $L(i+1)$，把 $R(i)-1$ 换成 $R(i+1)$，但这算出来就和 $G(i)$ 差了几个的 $f(n-i,a)$ 怎么办呢。先标记为 $\Delta$ 到后面再算呗。

$G(i)=\sum_{a=L(i+1)}^{R(i+1)+1}f(n-i,a)+\Delta$  
$=\sum_{a=L(i+1)}^{R(i+1)+1}(3f(n-(i+1),a-1)+2f(n-(i+1),a))+\Delta$  
$=3f(n-(i+1),L(i+1)-1)+2f(n-(i+1),L(i+1))+3f(n-(i+1),L(i+1))+ \cdots +2f(n-(i+1),R(i+1)))+3f(n-(i+1),R(i+1))+2f(n-(i+1),R(i+1)+1)+\Delta$  
$=5 \sum_{a=L(i+1)}^{R(i+1)}f(n-(i+1),a)+3f(n-(i+1),L(i+1)-1)+2f(n-(i+1),R(i+1)+1)+\Delta$  
$=5G(i+1)+3f(n-(i+1),L(i+1)-1)+2f(n-(i+1),R(i+1)+1)+\Delta$

#### 代码实现

在式子乘个 `1ll` 可以有效避免溢出的问题。计算高次幂为了避免溢出，在写快速幂的时候也要每步取模。计算组合数的时候需要用到除以比较大的数，还是为了避免溢出这时候也是要取模的。于是就打开了乘法逆元的新世界，道理其实也不是很懂，大概就是如果有一个素数 $p$，根据费马小定理则有 $a^{p-1}\equiv1(mod\;p)$，那么 ${a}\cdot a^{p-2}\equiv1(mod\;p)$，$a^{p-2}$ 就叫 $a\ mod\ p$ 意义下的逆元，照用就是了。

    #include <bits/stdc++.h>
    using namespace std;
     
    const int mod = 998244353;
    const int N = 3000010;
    int n, m, f2[N], f3[N], q[N], p[N], G[N];
     
    
    //////// 快速幂 ////////
    int qpow(int a, int b) {
        int ans = 1;
        a %= mod;
        for (; b; b >>= 1) {
            if (b & 1) ans = 1ll * ans * a % mod;
            a = 1ll * a * a % mod;
        }
        return ans;
    }
    
    //////// 组合数 ////////
    int c(int a, int b) { return 1ll * q[a] * p[b] % mod * p[a - b] % mod; }
    
    //////// 算 f(a,b) ////////
    int f(int a, int b) { return 1ll * c(a, b) * f3[b] % mod * f2[a - b] % mod; }
    
    int main() {
        cin >> n >> m;
        f2[0] = f3[0] = q[0] = 1;
        for (int i = 1; i <= n; i++) {
            // 算 i!，2 的 i 次方，3 的 i 次方，对 mod 取模
            q[i] = 1ll * q[i - 1] * i % mod;
            f2[i] = 1ll * f2[i - 1] * 2 % mod;
            f3[i] = 1ll * f3[i - 1] * 3 % mod;
        }
    
        // 乘法逆元
        p[n] = qpow(q[n], mod - 2);
        for (int i = n - 1; i >= 0; i--) p[i] = 1ll * p[i + 1] * (i + 1) % mod;
    
        //////// 算 G(i) ////////
        int l = 0, r = 1; G[n] = 1;
        for (int i = n - 1; i >= 0; i--, r++) {
    
            // 算 5 * G(i + 1) + 3 * f(...) + 5 * f(...)
            G[i] = (5ll * G[i + 1] % mod + 3ll * f(n - i - 1, l - 1) % mod + 2ll * f(n - i - 1, r) % mod) % mod;
            int ql = max((n - i - m + 1) / 2, 0), qr = min((n - i + m) / 2, n - i);
            // 此时 l = L(i + 1), r = R(i + 1) + 1
            // 此时 ql = L(i), qr = R(i)
            
            // 算 delta
            while (l < ql) G[i] = (1ll * G[i] - f(n - i, l) + mod) % mod, l++;
            while (l > ql) l--, G[i] = (1ll * G[i] + f(n - i, l)) % mod;
            while (r > qr) G[i] = (1ll * G[i] - f(n - i, r) + mod) % mod, r--;
            while (r < qr) r++, G[i] = (1ll * G[i] + f(n - i, r)) % mod;
            // 此时 l = L(i), r = R(i)
            // 此时 i--, r++
        }
    
        //////// 算答案 ////////
        int ans = 0;
        for (int i = n; i >= 0; i--) ans = (ans + 1ll * G[i] * c(n, i) % mod) % mod;
        cout << ans << endl;
        return 0;
    }

# 2020 牛客寒假算法基础集训营 5
 
> 字符串、二分、哈希、DP、模拟、搜索、数学

5A. 模板
-----

### 大意

给出两个字符串，允许对其中一个字符串做任意的替换、删除最后一位、在末尾添加一个字母。问最少的步数，使两个字符串相同。

### 思路

签到题。逐个位置匹配，发现不一样则步数加一。当长度较短的字符串匹配完后再加上两个字符串的长度差。

    #include <bits/stdc++.h>
    using namespace std;
     
    int main()
    {
        int n, m;
        cin >> n >> m;
        string a, b;
        cin >> a >> b;
        if (n < m)
        {
            swap(n, m);
            swap(a, b);
        }
        int ans = 0;
     
        for (int i = 0; i < m; i++)
        {
            if (a[i] != b[i])
                ans++;
        }
        ans += n - m;
        cout << ans << endl;
        return 0;
    }

5B. 牛牛战队的比赛地
-----------

### 大意

已知多个点的坐标，问平面上哪个位置满足到这些点的距离的最大值最小。

### 思路

三分答案。大概是因为它是二次函数，每次使用三分都能排除最差的一部分答案。二分答案似乎也是可以的，但二分的依据是单调性，二次函数也没法保证这一点，所以做起来似乎要麻烦很多。

    #include <bits/stdc++.h>
    using namespace std;
    
    struct p
    {
        int x, y;
    } a[100005];
    int n;
    double check(double x)
    {
        double max = 0;
        for (int i = 0; i < n; i++)
        {
            double tmp = sqrt(a[i].y * a[i].y + (a[i].x - x) * (a[i].x - x));
            if (tmp > max)
                max = tmp;
        }
        return max;
    }
    double tsearch(double left, double right)
    {
        int i;
        double mid, midmid;
        for (i = 0; i < 100; i++)
        {
            mid = left + (right - left) / 2;
            midmid = mid + (right - mid) / 2;
            if (check(mid) > check(midmid))
                left = mid;
            else
                right = midmid;
        }
        return mid;
    }
    int main()
    {
        cin >> n;
        for (int i = 0; i < n; i++)
            cin >> a[i].x >> a[i].y;
        double max = tsearch(-10000, 10000);
        cout << check(max) << endl;
        return 0;
    }

5C. C 语言 IDE
-----------

### 大意

输入一份 C 语言代码，要求输出代码中出现的函数。

### 思路

哦，是码农大模拟！爱了爱了。

    #include <bits/stdc++.h>
    using namespace std;
    
    string source;
    void replaceAll(string &s, string oldstr, string newstr)
    {
        for (string::size_type pos = 0; pos != string::npos; pos += newstr.length())
            if ((pos = s.find(oldstr, pos)) != string::npos)
                s.replace(pos, oldstr.length(), newstr);
            else
                break;
    }
    struct functions
    {
        string inClass, name, outputType;
        vector<string> inputType;
        functions(string inClass = "", string name = "", string outputType = "void", vector<string> inputType = vector<string>(0))
            : inClass(inClass), name(name), outputType(outputType), inputType(inputType) {}
    };
    vector<functions> funs;
    void solve(string &s)
    {
        replaceAll(s, "/*", " /* ");
        replaceAll(s, "*/", " */ ");
        replaceAll(s, "//", " // ");
        replaceAll(s, "(", " ( ");
        replaceAll(s, ")", " ) ");
        replaceAll(s, "{", " { ");
        replaceAll(s, "}", " } ");
        replaceAll(s, "=", " = ");
        replaceAll(s, "\"", " \" ");
        replaceAll(s, "'", " ' ");
        replaceAll(s, ";", " ; ");
        replaceAll(s, ",", " , ");
        replaceAll(s, "+ = ", "+=");
        replaceAll(s, "- = ", "-=");
        replaceAll(s, "* = ", "*=");
        replaceAll(s, "/ = ", "/=");
        replaceAll(s, "^ = ", "^=");
        replaceAll(s, "| = ", "|=");
        replaceAll(s, "& = ", "&=");
        replaceAll(s, ":", " : ");
        replaceAll(s, " :  : ", "::");
        vector<string> tokens;
        string now = "";
        for (int i = 0; s[i]; i++)
        {
            if (s[i] == ' ' || s[i] == '\t' || s[i] == '\r' || s[i] == '\n' || s[i] == '\0')
            {
                if (now != "")
                {
                    if (now == ":" && tokens.back() == ")")
                    {
                        string tmpnow = "";
                        for (int j = i + 1; s[j]; j++)
                        {
                            if (s[j] == ' ' || s[j] == '\t' || s[j] == '\r' || s[j] == '\n' || s[j] == '\0')
                            {
                                if (tmpnow == "{")
                                {
                                    now = "{";
                                    i = j - 1;
                                    break;
                                }
                                tmpnow = "";
                            }
                            else
                                tmpnow += s[j];
                        }
                        continue;
                    }
                    if (now == "const")
                    {
                        now = "";
                        continue;
                    }
                    if (now == "//")
                    {
                        for (int j = i; s[j]; j++)
                        {
                            if (s[j] == '\n')
                            {
                                i = j - 1;
                                break;
                            }
                        }
                        now = "";
                        continue;
                    }
                    if (now == "/*")
                    {
                        int num = 1;
                        string tmpnow = "";
                        for (int j = i + 1; s[j]; j++)
                        {
                            if (s[j] == ' ' || s[j] == '\t' || s[j] == '\r' || s[j] == '\n' || s[j] == '\0')
                            {
                                if (tmpnow == "/*")
                                    num++;
                                if (tmpnow == "*/")
                                {
                                    num--;
                                    if (num == 0)
                                    {
                                        i = j - 1;
                                        break;
                                    }
                                }
                                tmpnow = "";
                            }
                            else
                                tmpnow += s[j];
                        }
                        now = "";
                        continue;
                    }
                    tokens.push_back(now);
                    now = "";
                }
            }
            else
                now += s[i];
        }
        int cnt = 0;
        string nowNamespace = "";
        for (int i = 1; i < (int)tokens.size(); i++)
        {
            if ((tokens[i] == "struct" || tokens[i] == "class") && tokens[i + 2] == "{")
            {
                cnt = 0;
                nowNamespace = tokens[i + 1];
                i += 2;
            }
            functions tmp(nowNamespace);
            if (tokens[i] == "{" && tokens[i - 1] == ")")
            {
                int num = 1;
                for (int j = i - 2; j >= 0; j--)
                {
                    if (tokens[j] == ")")
                        num++;
                    if (tokens[j] == "(")
                    {
                        num--;
                        if (num == 0)
                        {
                            tmp.name = tokens[j - 1];
                            tmp.outputType = "";
                            for (int k = j - 2; k >= 0; k--)
                                if (tokens[k] != "}" && tokens[k] != "}" && tokens[k] != ";" &&
                                    tokens[k].back() != ':' && tokens[k] != "inline" &&
                                    tokens[k] != "static" && tokens[k][0] != '#' &&
                                    tokens[k].back() != '\"' && tokens[k].back() != '>')
                                    tmp.outputType = tmp.outputType == "" ? tokens[k] : tokens[k] + " " + tmp.outputType;
                                else
                                    break;
                            int last = i - 2;
                            for (int k = i - 2; k >= j; k--)
                            {
                                if (tokens[k] == "(" || tokens[k] == ",")
                                {
                                    string tt = "";
                                    for (int t = k + 1; t < last; t++)
                                        tt = tt == "" ? tokens[t] : tt + " " + tokens[t];
                                    if (tt != "")
                                        tmp.inputType.push_back(tt);
                                    last = k - 1;
                                }
                                if (tokens[k] == "=" || tokens[k] == ")")
                                    last = k - 1;
                            }
                            reverse(tmp.inputType.begin(), tmp.inputType.end());
                            break;
                        }
                    }
                }
                funs.push_back(tmp);
                num = 1;
                for (int j = i + 1; j < (int)tokens.size(); j++)
                {
                    if (tokens[j] == "{")
                        num++;
                    if (tokens[j] == "}")
                    {
                        num--;
                        if (num == 0)
                        {
                            i = j;
                            break;
                        }
                    }
                }
                continue;
            }
            if (nowNamespace != "")
            {
                if (tokens[i] == "{")
                    cnt++;
                if (tokens[i] == "}")
                {
                    cnt--;
                    if (!cnt)
                        nowNamespace = "";
                }
            }
        }
    }
    int main()
    {
        char ch;
        while ((ch = getchar()) != EOF)
            source += ch;
        solve(source);
        for (auto &i : funs)
        {
            if (i.outputType != "")
                cout << i.outputType << " ";
            if (i.inClass != "")
                cout << i.inClass << "::";
            cout << i.name << "(";
            for (int j = 0; j < (int)i.inputType.size(); j++)
                cout << i.inputType[j] << (j == (int)i.inputType.size() - 1 ? ")" : ",");
            if ((int)i.inputType.size() == 0)
                cout << ")";
            cout << endl;
        }
        return 0;
    }

5E. Enjoy the Game
-----------------

### 大意

给出一个规则：$n$ 张卡牌，先手第一步最少拿 $1$ 张，最多拿 $n-1$ 张。接下来每一步，双方最少要拿 $1$ 张，最多拿等同于上一步对方拿的牌数的牌。拿走最后一张牌胜。问对不同的 $n$，先手是否有必胜策略。

### 思路

找规律找出来的，只要是二的幂就是 Alice 胜，否则就是 Bob 胜。这题好像把 `__builtin_popcount(n)` 给卡了，原因不明，反正下次是不敢用了。

    #include <bits/stdc++.h>
    using namespace std;
     
    long long lowbit(long long x)
    {
        return x&(-x);
    }
       
    int main()
    {
        long long n;
        cin >> n;
        if (lowbit(n) == n)
        {
            cout << "Alice" << endl;
            return 0;
        }
        cout << "Bob" << endl;
        return 0;
    }

5H. Hash
-------

### 大意

给出一个字符串和一个 Hash 函数（核心代码 `res = (res * 26 + str[i] - 'a') % mod;`），求一个字典序最小且大于该字符串且有着相同 Hash 的字符串。

### 思路

其实就是把字符串转成了二十六进制数，然后给它取模。那么要想 Hash 值相同字典序还要大于原字符串，就直接给这个二十六进制数加上模数就好了。

    #include <bits/stdc++.h>
    using namespace std;
     
    int main()
    {
        string a;
        int m;
        int ta[6];
        while (cin >> a >> m)
        {
            int org[6], backup[6];
            for (int i = 0; i < 6; i++)
            {
                org[i] = a[i] - 'a';
                backup[i] = org[i];
            }
            string b = a;
            for (int i = 5; i >= 0; i--)
            {
                ta[i] = m % 26;
                m /= 26;
            }
            if (m != 0)
            {
                cout << "-1" << endl;
            }
            for (int i = 5; i > 0; i--)
            {
                org[i] += ta[i];
                if (org[i] > 25)
                {
                    org[i] -= 26;
                    org[i - 1]++;
                }
            }
            org[0] += ta[0];
            if (org[0] < 26)
            {
                for (int i = 0; i < 6; i++)
                {
                    printf("%c", org[i] + 'a');
                }
                cout << endl;
                continue;
            }
            else
            {
                cout << "-1" << endl;
                continue;
            }
        }
     
        return 0;
    }

5I. I 题是个签到题
-----------

### 大意

给出一场比赛参赛人数和各题过题人数，通过人数不低于全场人数的 $80\%$ 或在所有题目中前三多就叫签到题。问 I 题是不是签到题。

### 思路

签到题。排序即可，关键是要处理并列的情况。

    #include <bits/stdc++.h>
    using namespace std;
     
    struct temp
    {
        int num;
        int ac;
    } a[20000];
     
    bool cmp(temp a, temp b)
    {
        return a.ac > b.ac;
    }
     
    int main()
    {
        int n, m;
        cin >> n >> m;
        for (int i = 0; i < n; i++)
        {
            cin >> a[i].ac;
            a[i].num = i;
        }
        if (a[8].ac * 10 >= m * 8)
        {
            cout << "Yes" << endl;
            return 0;
        }
        sort(a, a + n, cmp);
        int cnt = 1;
        int cur = 1;
        if (a[0].num == 8)
        {
            cout << "Yes" << endl;
            return 0;
        }
        for (int i = 1; i < n; i++)
        {
            if (a[i].ac != a[i - 1].ac)
            {
                cnt += cur;
                cur = 1;
            }
            else
            {
                cur++;
            }
            if (a[i].num == 8 && cnt <= 3)
            {
                cout << "Yes" << endl;
                return 0;
            }
            if (cnt > 3)
            {
                cout << "No" << endl;
                return 0;
            }
        }
    }

5J. 牛牛战队的秀场
----------

### 大意

求圆内接正 $n$ 边形的边长。

### 思路

签到。可以很容易地计算每条弦对应的圆心角的大小，然后三角函数。

    #include <bits/stdc++.h>
    using namespace std;
     
    #define PI 3.1415926535898
     
    int main()
    {
        double n, r, i, j;
        cin >> n >> r >> i >> j;
        if (i == j)
        {
            cout << "0" << endl;
            return 0;
        }
        if (i > j)
            swap(i, j);
        double ans = min(j - i, i - j + n);
        printf("%.8lf\n", ans * 2 * r * sin(360 / n / 2 * PI / 180));
        return 0;
    }

# 2020 牛客寒假算法基础集训营 6

> 贪心、图论、构造、二分、计数、数论、思维

6A. 配对
-----

### 大意

给出两个集合，每个集合里有 $N$ 个数，不同集合的两个数配对并求和，要求最大化第 $K$ 大的和。

### 思路

    #include <bits/stdc++.h>
    using namespace std;
     
    typedef long long ll;
     
    int main()
    {
        ll n, k;
        cin >> n >> k;
        ll a[n], b[n], c[n];
        for (ll i = 0; i < n; i++)
        {
            cin >> a[i];
        }
        for (ll i = 0; i < n; i++)
        {
            cin >> b[i];
        }
        sort(a, a + n);
        sort(b, b + n);
        for (int i = 0; i < k; i++)
        {
            c[i] = a[n - 1 - i] + b[n - k + i];
        }
        sort(c, c + k);
        cout << c[0] << endl;
        return 0;
    }

B.
--