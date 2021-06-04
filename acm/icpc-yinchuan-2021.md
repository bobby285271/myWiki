---
title: ICPC Yinchuan Regional 2021
description: 
published: true
date: 2021-06-04T04:23:29.314Z
tags: 
editor: markdown
dateCreated: 2021-06-04T04:23:29.314Z
---

记录一下银川字典树题的 AC 代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

ll n;
vector<int> node[2500005];
int tot = 1;
char let[2500005];
bool rel[2500005];
string str[50005];
int tmp[2500005];

void build_tree(string &s, int ind, int rt)
{
    if (ind == s.size())
        return;
    int siz = node[rt].size();
    int flag = -1;
    for (int i = 0; i < siz; i++)
    {
        if (s[ind] == let[node[rt][i]])
        {
            flag = i;
            break;
        }
    }
    if (flag == -1)
    {
        node[rt].push_back(tot);
        let[tot] = s[ind];
        tot++;
        build_tree(s, ind + 1, tot - 1);
    }
    else
    {
        build_tree(s, ind + 1, node[rt][flag]);
    }
}

void dfs_solve(string s, int ind, int rt)
{
    if (ind == s.size())
    {
        rel[rt] = 1;
        tmp[rt] = 0;
        return;
    }
    int siz = node[rt].size();
    bool flag = 0;
    int num = 0;
    int co = 0;
    for (int i = 0; i < siz; i++)
    {
        if (s[ind] == let[node[rt][i]])
        {
            dfs_solve(s, ind + 1, node[rt][i]);
        }
        if (rel[node[rt][i]] == 1)
        {
            int tnum = tmp[node[rt][i]];
            num += tnum;
            if (tnum == 0)
                co++;
        }
        else
        {
            flag = 1;
        }
    }
    if ((flag == 1 || num > 0) && co > 0)
    {
        num += co;
    }
    rel[rt] = 1;
    tmp[rt] = num;
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> str[i];
        build_tree(str[i], 0, 0);
    }
    for (int i = 0; i < n; i++)
    {
        int ans = 0;
        int siz = node[0].size();
        for (int j = 0; j < siz; j++)
        {
            if (str[i][0] == let[node[0][j]])
                dfs_solve(str[i], 1, node[0][j]);
            if (rel[node[0][j]] == 1)
            {
                int tnum = tmp[node[0][j]];
                if (tnum == 0)
                    tnum = 1;
                ans += tnum;
            }
        }
        cout << ans << endl;
    }
}
```