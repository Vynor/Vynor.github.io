# day5D背包问题 # 

### 题目描述 ###
共 $n$ 件物品，第 $i$ 件物品的价值是 $a_i$，
小A决定在这 $n$ 件物品中挑选出价值和是 $m$ 的物品装进背包里带给小B。
但是糟糕的是背包不慎破了一个洞，有一些物品(也有可能是0或全部)遗失了。
小A现在想知道剩下的价值和可能是多少？(升序输出)
#### 数据范围 ####
$n,m,a_i \le 500$

## 思路 ##
我们考虑对于一件物品，他只有这3种情况：
- 被装进包里，但在途中遗弃
- 被装进包里，并成功带给对方
- 没有被选择装进包里

这样搜索的时间就是 $O(3^n)$。

我们设 $f_{i,j,k}$表示，考虑前 $i$ 个物品，小A装进背包的价值和是 $j$，交付小B的价值和是 $k$，这样的方案是否存在。

考虑转移，对于一个物品 $w$ :
- 被装进包里，但在途中遗弃: $f_{i,j,k}$ <- $f_{i-1,j-w,k}$
- 被装进包里，并成功带给对方: $f_{i,j,k}$ <- $f_{i-1,j-w,k-w}$
- 没有被选择装进包里: $f_{i,j,k}$ <- $f_{i-1,j,k}$

注意 $500^3$ 的空间会爆，可以考虑滚动数组或2维空间优化

## 代码 ##
```c++
const int N = 5e3 + 10;

int n, m;
int f[N][N];

void solve() {
    cin >> n >> m;
    f[0][0] = 1;
    for (int i = 1; i <= n; i ++ ){
        int w; 
        cin >> w;
        for (int j = m; j >= 0; j -- )
            for (int k = j; k >= 0; k -- ){
                if (j >= w)
                    f[j][k] |= f[j - w][k];
                if (j >= w && k >= w)
                    f[j][k] |= f[j - w][k - w];
                //f[j][k] |= f[j][k]
            }
    }
    for (int k = 0; k <= m; k ++ )
        if (f[m][k])
            cout << k << ' ';
}
```
# day4F配送 #

### 题目描述 ###
有1张 $n$ 个顶点 和 $m$ 条边的无向图，每条边具有通行时间。问 $1-n$ 的路径的权值最小是多少？一条路径的权值定义为通过边的最长通行时间和次长通行时间的和。
#### 数据范围 ####
$2 \le n \le 3e5, m \le 1e6, 1 \le u,v \le n, 1 \le w \le 1e9$

## 思路 ##
#### 二分 ####
考虑二分答案ans，设置 $d_x$表示从 $1-x$ 的**最大通行时间**。走一遍最短路，如果 $d_y + w \le ans$，表示可以从 $x$ 走到 $y$，否则超过二分值，不可走，最后返回**是否**可以到 $n$。
时间复杂度为 $O({log{2e9}} *  MlogM)$，显然会TLE，二分做法不可取

#### 枚举 ####
考虑**枚举边**作为 $1-n$ 路径上**最大通行时间** $MAX$ 的贡献者。那么只需要求出路径上**除去**该边的**次大值** $subMAX$ 即可，分别求出 $1$ 到 该边的一个端点的最大值 和 该边的另外一个端点到 $n$ 的最大值，2者取 $max$ 就是次大值。

设置该边的2个端点分别为 $x,y$，
- $1-x$ 的最大通行时间可以参照二分思路中的 $check$ 求解
- 而对于 $y-n$ 的最大通行时间，因为 $y$ 常变 $n$ 唯一，所以可以预处理 $n-y$ 的最短路下的最大通行时间。因为是无向图，所以 $n-y = y-n$。

考虑转移，
$subMAX = min(max(d1_x, dn_y),max(d1_y, dn_x))$
如果 $MAX \geq subMAX$, $ans = max(ans, MAX + subMAX)$

```c++
using namespace std;
const int N = 1e6 + 10;

struct Node{
    int x, y, v;
}E[N];

vector<pair<int, int>> edge[N];

int n, m;
int d1[N], dn[N];
bool b[N];

using pt = pair<int, int>;

void dij(int st, int *dis){
    for (int i = 1; i <= n; i ++ ) b[i] = 0, dis[i] = 1e9;
    dis[st] = 0;
    priority_queue<pt, vector<pt>, greater<pt>> q;
    q.push({dis[st], st});
    while (!q.empty()){
        int x = q.top().second;
        q.pop();
        if (b[x]) continue;
        b[x] = 1;
        for (auto [y, v]:edge[x])
            if (dis[y] > max(dis[x], v)){
                dis[y] = max(dis[x], v);
                q.push({dis[y], y});
            }
    }
}

void solve() {
    cin >> n >> m;
    for (int i = 1; i <= m; i ++ ){
        int x, y, v;
        cin >> x >> y >> v;
        E[i] = {x, y, v};
        edge[x].push_back({y, v});
        edge[y].push_back({x, v});
    }
    dij(1, d1);
    dij(n, dn);
    int ans = 2e9;
    for (int i = 1; i <= m; i ++ ){
        auto [x, y, MX] = E[i];
        int sub = min(max(d1[x], dn[y]), max(d1[y], dn[x]));
        if (MX >= sub){
            ans = min(ans, MX + sub);
        }
    }
    cout << ans;
}
```
# day4G字符串替换 # 

### 题目描述 ###
给定一个长度为 $n$ 的字符串 $s$，仅包含 $a,b,c,?$ 4种字符。
你可以将字符 $$