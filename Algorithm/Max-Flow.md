---
Author: HammerLi
---

[TOC]

# 最大流问题的算法分析及代码实现

## 摘要

在本次课程中，给我留下最深印象的问题是最大流问题。我认为，有向图所构成的网络是交通物流方面建模的不错选择，但在解决基于该模型下的最优解问题上我所了解的精确算法不多，而相关的启发式算法倒是了解了不少，比如我在大一的人工智能导论课的结课作业中针对VRP问题（即车辆调度问题）探究了一种启发式算法——遗传算法。因此针对网络流中的最大流问题，我想更加深入地学习一下有向图的基本建模和相关的精确算法，本文也将基于这两点组织内容，而鉴于我的水平有限，相关定理和结论的证明从略，同时最后会实际编程实现算法来解决特定问题。

## 问题描述及相关概念

### 流、流网络及最大流问题

流网络是一个有向连通图$G=<V,E>$，其中每条边$e(v,u)\in E$都具有一个容量值也就是“管道”大小$c(v,u)$，满足$c(v,u)>0$，同时要求若$e(v,u)\in E$，则不存在$e'(u,v)\in E$，即不存在反向边。在图$G$中需要指出两个重要的结点——源结点$s$和汇集点$t$。

因为图$G$是连通有向图，因此，对于除$s,t$两点之外的结点$v$，从源结点到汇集点的路径中一定存在一条经过结点$v$的路径，也就是说对于除源结点外每一个结点的入度大于$0$，则我们有$|E|\ge|V|-1$。

进而我们在上述流网络中形式化定义流：

> 流是一个实值函数$f:V\times V\rightarrow \bold R$，满足以下两条性质：
> $$
> \begin{align}
> (1)\ &\forall u,v\in V,0\le f(u,v)\le c(u,v)\\
> (2)\ &\forall u\in V-\{s,t\},\sum_{v\in V}f(v,u)=\sum_{v\in V}f(u,v)(\forall (u,v)\notin E,f(u,v)=0)
> \end{align}
> $$
> 我们称非负数值$f(u,v)$为从结点$u$到$v$的流，其值$|f|$如下定义：
> $$
> |f|=\sum_{v\in V}f(s,v)-\sum_{v\in V}f(v,s)
> $$

那么最大流问题顾名思义，就是在给定流网络$G$，以及一个源结点$s$，一个汇集点$t$以及各边容量的条件下找到值最大的一个流。这里需要说明的是，针对存在多个源结点和汇集点的情况以下不做讨论，但他们都可以通过一些等效手段转换为上述的简单模型。

### 残存网络和增广路径

为了方便描述算法，我们引入两个名词——残存网络$G_f$和增广路径$p$。

残存网络是由那些能够对流量进行调整的边构成的另一个有向图。为了获取仍有余量可以增加的边，我们对$G$中各边计算$c_f(v,u)=c(v,u)-f(v,u)$，若$c_f(v,u)>0$，则$e(v,u)\in G_f$。同时为了表示对一条边上的流量进行削减操作，我们可以将具有正流量的边对应的反向边加入$G_f$，即有$c_f(u,v)=f(v,u)$，也就是说该边能够允许的反向流量最多和这条边抵消。综合以上我们有：
$$
c_f(v,u)=\begin{cases}
c(v,u)-f(v,u)&(v,u)\in E\\
f(v,u)&(u,v)\in E\\
0&other\ cases
\end{cases}
$$
按照这样的定义，我们由流$f$在网络流$G$中诱导出了另一个图——残存网络$G_f=<V,E_f>$，其中$E_f$满足$E_f=\{ (u,v)\in V\times V:c_f(u,v)>0 \}$。

从上面的定义来看，残存网络显示了整个网络流的提升潜能，所以我们需要借助它来对流$f$进行优化。显然残存网络也同样满足网络流的性质，因此我们可以在其中确定一个流$f'$，进一步，我们利用$f'$对$f$进行递增操作，如下：
$$
(f'*f)(u,v)=\begin{cases}
f(u,v)+f'(u,v)-f'(v,u)&(u,v)\in E\\
0&\text{other cases}
\end{cases}
$$
增广路径$p$是残存网络$G_f$中一条从源结点$s$到汇集点$t$的简单路径。根据以上残存网络的定义，对于增广路径上的一条边$(u,v)$，它可以被增加的流量幅度最大为$c_f(u,v)$。进一步我们考虑整个路径所能增加的流量幅度，显然这一结果为整个路径中残存余量的最小值，因此我们有如下表达：
$$
c_f(p)=\min \{c_f(u,v):(u,v)\in p \}
$$

### 流网络中的割与最大流最小割定理

流网络$G=<V,E>$的割$(S,T)$将$V$划分成$S$和$T=V-S$两个部分，使得$s\in S,t\in T$。割的容量记为$c(S,T)$。一个网络中所有割中具有最小容量的割被称作最小割。接下来不加证明地给出最大流最小割定理：

> 如果$f$是具有源结点$s$和汇集点$t$的流网络$G=<V,E>$的一个流，则有如下三个等价说法：
>
> 1. $f$是$G$中的一个最大流
> 2. 残余网络$G_f$不含增广路径
> 3. 对于$G$中的最小割$(S,T)$，有$|f|=c(S,T)$ 

### 层次图

在流网络$G$中考虑各个结点到源结点$s$的距离对结点进行分层所获得的图称为层次图。

## 解决最大流问题的多种算法

### 统一输入和输出

- 算法输入：网络流$G$（要求涵盖点，边，反向边，容量四方面信息）、源节点和汇集点。
- 算法输出：该网络流中最大流的值。

### Edmonds-Karp算法

1. 初始化，输入边，相应容量。
2. 从源节点对网络流进行广度优先搜索，寻找最短的增广路径：
   1. 若不存在，则说明当前流不需要更新，最大流已经出现，算法结束；
   2. 若存在，则依据寻找到的增广路径对当前流进行递增操作，进行下一轮的广度优先搜索操作。

### Dinic算法

1. 初始化，将流中的边全置零。
2. 对残余网络进行广度优先搜索，进而构造层次图：
   1. 若汇集点不存在层次，说明不存在层次图，也就不存在增广路径，最大流已经出现，算法结束。
   2. 若汇集点的层次存在，则利用深度优先搜索从第一层开始逐层向后反复寻找增广路径：
      1. 若寻找到增广路径，则更新当前流。
      2. 若没有找到增广路径，则返回第二步，构造新的层次图。

### ISAP算法

1. 初始化。
2. 利用广度优先搜索计算结点的标号即该结点到源结点的最短距离。
3. 循环对当前结点连接的可行边（即邻接边）进行寻找增广路径：
   1. 若找到增广路径，则进行递增操作。
   2. 若没有找到增广路径，则说明最大流已经出现，算法结束。
4. 更新当前结点的标号，保证可行边的存在以及搜索路径的最小值，继续进行下一步循环搜索。

### 定性分析

对于Edmonds-Karp算法来说，仅仅是将朴素算法中寻找增广路径的过程优化成了广度优先搜索，从而排除了选择更加“绕”的路径的低效情形。

Dinic算法则是针对循环广度优先搜索中进行增广的次数作了一定的优化，它依据图中结点到源结点的距离进行分层，利用深度优先搜索在逐层进行广度优先搜索的同时寻找多条增广路径。尽管渐进时间复杂度没有改变，但实际上其系数有一定的削减，在以下的测试结果中我们可以看到这一点。

ISAP算它省去了Dinic每次增广后需要重新构建分层图的麻烦，而是在每次增广完成后自动更新每个点的标号（也就是所在的层），降低了广度优先搜索的次数，进一步优化了算法。

在不使用高级数据结构的优化下，三者的渐进时间复杂度均为$O(|V|\cdot|E|^2)$，但实际上因为算法思想的不同，其速度是递增的。

## 算法实现与测试结果

以上算法采用最朴素的方法进行代码（C++）实现，并进行算例测试，主要代码如下，详细代码及样例见附录。

采用的样例大小为10000个结点，100000条边的网络流，测试结果如下（时间单位：秒）：

![]()

Edmonds-Karp算法测试结果

![]()

Dinic算法测试结果

![]()

ISAP算法测试结果

实验结果和我们预期的一样。但由于时间有限，我所找到的数据样例只有一组，故存在偶然性，后期仍需采用大量数据进行更多测试。

## 附录

### Edmonds-Karp算法

```c++
#include <iostream>
#include <cstring>
#include <queue>
#include <algorithm>

using namespace std;

const int inf = 0x3f3f3f3f;
const int maxn = 10005; // max number of vertex
int vn, en;             // vertex number and edge number
int g[maxn][maxn];      // the adjacent matrix of graph
int pre[maxn];          // the previous node to the current node i
int f[maxn];            // the flow from starting node to the cur node

int bfs(int s, int t)
{
    queue<int> q;
    memset(pre, -1, sizeof(pre));

    q.push(s);
    pre[s] = 0;
    f[s] = inf;

    while (!q.empty())
    {
        int cur = q.front();
        q.pop();
        if (cur == t)
            break;
        for (int i = 1; i <= vn; ++i)
        {
            if (i != cur && g[cur][i] > 0 && pre[i] == -1)
            {
                pre[i] = cur;
                f[i] = min(g[cur][i], f[cur]);
                q.push(i);
            }
        }
    }

    if (pre[t] != -1)
        return f[t];
    else
        return -1;
}

int EK(int s, int t)
{
    int maxFlow = 0, inc = 0;
    while ((inc = bfs(s, t)) != -1)
    {
        int k = t;
        while (k != s)
        {
            int last = pre[k];
            g[last][k] -= inc;
            g[k][last] += inc;
            k = last;
        }
        maxFlow += inc;
    }
    return maxFlow;
}

int main(void)
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    int s, t;
    cin >> vn >> en >> s >> t;
    int bp, ep;
    int w;
    for (int i = 0; i < en; ++i)
    {
        cin >> bp >> ep >> w;
        g[bp][ep] += w;
    }
    clock_t start = clock();
    cout << "The maximum flow is ";
    cout << EK(s, t) << endl;
    clock_t end = clock();
    cout << "The time cost is " << (double)(end - start) / CLOCKS_PER_SEC << endl;
    return 0;
}
```

### Dinic算法

```c++
#include <iostream>
#include <cstring>
#include <algorithm>
#include <ctime>
using namespace std;

const int maxn = 10005, maxm = 200005;
int cnt = 1, h[maxn];
int nxt[maxm], to[maxm], cap[maxm];

inline void addedge(int u, int v, int w)
{
    to[++cnt] = v;
    cap[cnt] = w;
    nxt[cnt] = h[u];
    h[u] = cnt;
}

int dep[maxn], q[maxn], l, r;

bool bfs(int s, int t, int n)
{
    memset(dep, 0, (n + 1) * sizeof(dep[0]));
    l = r = 1;
    q[r] = s;
    dep[s] = 1;
    while (l <= r)
    {
        int u = q[l++];
        for (int p = h[u]; p != 0; p = nxt[p])
        {
            int v = to[p];
            if (cap[p] > 0 && dep[v] == 0)
            {
                dep[v] = dep[u] + 1;
                q[++r] = v;
            }
        }
    }
    return dep[t];
}

int dfs(int u, int t, int in)
{
    if (u == t)
        return in;
    int out = 0;
    for (int p = h[u]; p != 0 && in != 0; p = nxt[p])
    {
        int v = to[p];
        if (cap[p] > 0 && (dep[v] == dep[u] + 1))
        {
            int res = dfs(v, t, min(cap[p], in));
            cap[p] -= res;
            cap[p ^ 1] += res;
            in -= res;
            out += res;
        }
    }
    if (out == 0)
        dep[u] = 0;
    return out;
}
int dinic(int s, int t, int vn)
{
    int maxFlow = 0;
    while (bfs(s, t, vn))
        maxFlow += dfs(s, t, maxn);
    return maxFlow;
}
int main(void)
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    int vn, en, s, t;
    cin >> vn >> en >> s >> t;
    int bp, ep;
    int w;
    for (int i = 0; i < en; ++i)
    {
        cin >> bp >> ep >> w;
        addedge(bp, ep, w);
        addedge(ep, bp, 0);
    }
    clock_t start = clock();
    cout << "The maximum flow is ";
    cout << dinic(s, t, vn) << endl;
    clock_t end = clock();
    cout << "The time cost is " << (double)(end - start) / CLOCKS_PER_SEC << endl;
    return 0;
}
```

### ISAP算法

```c++
#include <bits/stdc++.h>
using namespace std;
#define inf 1000000010
int n, m, s, t;
struct Edge
{
    int v, w, next;
} e[300005];
int head[300005], tot = 1;
void add(int u, int v, int w)
{
    tot++;
    e[tot].v = v;
    e[tot].w = w;
    e[tot].next = head[u];
    head[u] = tot;

    tot++;
    e[tot].v = u;
    e[tot].w = 0;
    e[tot].next = head[v];
    head[v] = tot;
}
int gap[300005], d[300005], cur[300005];
queue<int> que;
void init(int s, int t)
{
    d[t] = 1, gap[1]++;
    for (int i = 1; i <= n; i++)
        cur[i] = head[i];
    que.push(t);
    while (!que.empty())
    {
        int x = que.front();
        que.pop();
        for (int i = head[x]; i; i = e[i].next)
        {
            if (!d[e[i].v])
            {
                d[e[i].v] = d[x] + 1;
                gap[d[x] + 1]++;
                que.push(e[i].v);
            }
        }
    }
}
int dfs(int x, int s, int t, int flow)
{
    if (x == t)
        return flow;
    int flowed = 0;
    for (int &i = cur[x]; i; i = e[i].next)
    {
        if (d[x] == d[e[i].v] + 1)
        {
            int tmp = dfs(e[i].v, s, t, min(flow, e[i].w));
            flowed += tmp, flow -= tmp, e[i].w -= tmp, e[i ^ 1].w += tmp;
            if (!flow)
                return flowed;
        }
    }
    gap[d[x]]--;
    if (!gap[d[x]])
        d[s] = n + 1;
    d[x]++;
    gap[d[x]]++;
    cur[x] = head[x];
    return flowed;
}
int ISAP(int s, int t)
{
    init(s, t);
    int ret = dfs(s, s, t, inf);
    while (d[s] <= n)
        ret += dfs(s, s, t, inf);
    return ret;
}
int main(void)
{
    scanf("%d%d%d%d", &n, &m, &s, &t);
    for (int i = 1; i <= m; i++)
    {
        int u, v, w;
        scanf("%d%d%d", &u, &v, &w);
        add(u, v, w);
    }
    clock_t start = clock();
    cout << "The maximum flow is ";
    cout << ISAP(s, t) << endl;
    clock_t end = clock();
    cout << "The time cost is " << (double)(end - start) / CLOCKS_PER_SEC << endl;
    return 0;
}
```

## 参考资料

[1] T. H. Cormen, *Introduction to Algorithms.* (3rd ed.) Cambridge, Mass: MIT Press, 2009.

[2] Edmonds–Karp algorithm Wikipedia, the free encyclopedia[EB/OL].2019.[https://en.wikipedia.org/wiki/Edmonds%E2%80%93Karp_algorithm](https://en.wikipedia.org/wiki/Edmonds–Karp_algorithm)

[3] Dinic's algorithm Wikipedia, the free encyclopedia[EB/OL].2019.[https://en.wikipedia.org/wiki/Dinic%27s_algorithm](https://en.wikipedia.org/wiki/Dinic's_algorithm)

