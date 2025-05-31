# ***Cost***

[SPOJ.com - Problem KOICOST](https://www.spoj.com/problems/KOICOST/)

这题绝对是一道好题，大致意思是这样的：给你一张图，定义$cost(u, v)$为删除最小边的权值，如果顶点 u 和 v 之间存在路径，则删除权重最小的边。Cost(u,v) 是此过程中删除的边的权重之和。

显然要求：$\sum_{1 <= u，v <= N} cost(u, v)$，显然，一个很自然的想法，我枚举所有的$u,v$累加所有符合要求的边权$w$，看一眼数据量，超时了，考虑优化。



正难则反，考虑逆向思维，我们能不能初始化一张空图，把边权从大到小加入，定义$P$为当我们要加入一条边$w$时，考虑$w$联通点的个数，此时的$w$所联通的点正好对应原题中删掉最小权重边所联通的点的个数，我们联通每一个$w$，联通两个点时的$cost(u, w)$加到$S$中，当联通所有点时，$S$即为答案。

$$
\begin{align}
&P = \sum_{每一个联通分量C}\begin{pmatrix} |C| \\ 2\end{pmatrix}\\
&S = 0\\
&S += S * P\:mod\: 1e9\\
&P +=  size(C_{u}) * size(C_{v})
\end{align}

$$

```cpp
#include<bits/stdc++.h>
using namespace std;
using ll = long long;

const int mod = 1e9;
const int N = 1e5 + 5;
ll n, m, pa[N], s[N];            //记着开long long 不然会WA
struct e {int u, v, w;};
int find(int x) {return x == pa[x] ? x : pa[x] = find(pa[x]);}
pair<bool, ll> union_(int x, int y) {
    x = find(x), y = find(y);
    if(x == y) return {false, 0};
    if(s[x] < s[y]) swap(x, y);
    ll c = (ll) (s[x] * s[y]);
    s[x] += s[y];
    pa[y] = x;
    return {true, c}; 
}
int main() {
    cin >> n >> m;
    vector<e> g(m + 1);
    for(int i = 1; i <= m; i++) {
        cin >> g[i].u >> g[i].v >> g[i].w;
    }
    for(int i = 1; i <= n; i++) {
        pa[i] = i;
        s[i] = 1;
    }
    sort(g.begin() + 1, g.end(), [](auto &x, auto &y) {
        return x.w > y.w;
    });
    // for(int i = 1; i < g.size(); i++) {
    //     cerr << g[i].u << ' ' << g[i].v  << ' ' << g[i].w << endl;
    // }
    ll P = 0;
    ll S = 0;
    for(int i = 1; i <= m; i++) {
        auto[mer, a] = union_(g[i].u, g[i].v);
        //cerr << S << ' ' << P << endl;
        if(mer) {
            P += a;
        }
        S = (S + (ll)g[i].w * P) % mod;
    }
    cout << S << endl;
    return 0;
}
```

洛谷
