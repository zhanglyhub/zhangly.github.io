$Prim$算法在最小生成树中是一种简单且易于实现的算法它不同于K算法加边的操作，采用小根堆的结构，每次加点。  
其实和$Dijkstra$算法一样，选择距离最小的一个节点，用新的边来更新到其他节点的距离。  

-------------------------------------  

下面提供证明：  
先来回顾一下$prim$算法的流程，以任意顶点r开始，设加入的点集为$S$，边集为$A$。  
每次选择$S$和$V╱S$的最小边权$e$加入$A$，并将$e$未连接的点加入$A$。  
当$V == S$ （所有点选完），算法结束  

那么，我们只需要证明，在算法每一步，p算法所选择的边ei组成成的边集$A∩T=A$。  
1）$A == ∅$，显然成立，$∅∈T$。  
2）$A≠∅，$假设算法执行到第$k$步时，$e∪A$依然$∈T$，根据割性质，则$k+1$步必然成立，$A$必然是$T$的子集。  

下面证明割性：  
对于图$（s.v/s），$必有横跨该割的最小边权$e$，一定属于这颗$MST$。  
反证法：假设原命题不成立，即$MST$中存在一颗树$T$，有$e∉MST$。  
将e加入T中，则T形成环，T中必然有一条边f横跨（s，v/s）否则无法联通，  
构造新树，T' = T∪e/f，断开f会断开T，加入e又重新连接，所以T'联通。  
权值，因为e<=f，w（T'）<= w（T）  
显然不成立，所以原命题成立



































------------------------------------------------------------------------------------------------------------------------------

## ***反向索引堆优化版***

在优化的版本中，完全不再依赖边， 只用维护点的信息即可，我把这种操作称为“解锁”，其实，和$Dijkstra$ 算法及其相似， 任意一个点：

1）当前节点在堆中， 考虑更新

2）当前节点不在堆，是新解锁的节点加入堆中

3）加入后调堆，结算答案，弹出堆顶元素。

核心思想就是维护一个索引数组，来记录堆中元素的位置

```cpp
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<iostream>
#include<cassert>
#define inf 1 << 30
using namespace std;

const int N = 5005;
const int M = 2e5 + 100;
int n, m, x, y, l, h[N], tot, dis[N], pos[N], q[N], sz;
bool vis[N];
struct e {
    int to, w, nxt;
}g[M << 1];
void add(int u, int v, int w) {         //链式前向星
    g[++tot].to = v;
    g[tot].w = w;
    g[tot].nxt = h[u];
    h[u] = tot; 
}
void up(int i) {        //索引堆
    while(i > 1) {
        int pa = i / 2;
        if(dis[q[i]] < dis[q[pa]]) {
            swap(q[i], q[pa]);
            pos[q[i]] = i;
            pos[q[pa]] = pa;
            i = pa;
        } else {
            break;
        }
    }
}
void down(int i) {
    while(1) {
        // int a = 0;a++;
        // if(a == 10) {cerr << a << ' '; break;}
        int l = i * 2, r = i * 2 + 1, best = i;
        if(l <= sz && dis[q[l]] < dis[q[best]]) best = l;
        if(r <= sz && dis[q[r]] < dis[q[best]]) best = r;
        if(best == i) break;
        swap(q[best], q[i]);
        pos[q[best]] = best;
        pos[q[i]] = i;
        i = best;
    }
}
int pop_() {
    int u = q[1];
    vis[u] = 1;
    q[1] = q[sz];
    pos[q[1]] = 1;
    sz--;
    //cerr << sz << endl;
    down(1);
    //cerr << "pop node " << u << ", dis=" << dis[u] << ", sz=" << sz << "\n";
    return u;
}
int main() {
    cin >> n >> m;
    memset(h, -1, sizeof(h));
    memset(vis, 0, sizeof(vis));
    for(int i = 1; i <= m; i++) {
        cin >> x >> y >> l;
        add(x, y, l);
        add(y, x, l);
    }
    // for(int i = 1; i <= tot; i++) {
    //     cout << g[i].to << ' ' << g[i].w << ' ' << g[i].nxt << endl;
    // }
    //cerr << "vvv" << endl;
    for(int i = 1; i <= n; i++) {
        pos[i] = i;
        q[i] = i;
        dis[i] = inf;
    }
    dis[1] = 0;
    sz = n;
    //up(pos[1]);
    int ans = 0, cnt = 0;
    //cerr << "bb" << endl;
    while(sz > 0) {
        int u = pop_();
        //cerr << u << ' ';
         if(dis[u] == inf) break;//如果弹出后节点边权inf，说明图从一开始就不连通直接break
        ans += dis[u];
        // cerr << ans << " ";
        cnt++;
        //cerr << ans << " " << cnt << endl;
        for(int i = h[u]; ~i; i = g[i].nxt) {
            int w = g[i].w, v = g[i].to;
            //cerr << v << ' ' << vis[v] << ' ' << dis[v] << ' ' << w << endl;
            if(!vis[v] && dis[v] > w) {
                dis[v] = w;
                up(pos[v]);
            }
        }
    }
    //cout << "kkkk" << endl;
    if(cnt < n) cout << "orz" << endl;
    else cout << ans << endl;
    return 0;
}
```


