[补题链接](https://codeforces.com/gym/104821)

# I. Counter # 
询问操作合法性
只要连续二个状态之间能相互转移，那么操作合法。
证明：状态固定，如果连续2个能相互转移，那么不连续的操作，一定存在前状态先到中间状态，然后实现后状态。
设置 $cnt = a_j - a_i, x, y$ 表示经过 $cnt$ 步从 $x$ 到 $y$
显然 $x,y$ 只与**最后一次**清零的位置有关，与清零次数无关
分类讨论：
1. 中途不清零
2. 中途清零一次

合法条件就是 $cnt == y - x \lor cnt - 1 >= y$

# F. Equivalent Rewriting #
观察到一个位置上元素的最终状态只与**最后一次**改变有关，操作间具有依赖关系。
对操作建图，合法操作就是拓扑序。图上一定存在 $1-n$ 拓扑序，因为任意一次操作只会对后面造成影响——连单向边。
新问题就是是否存在不同的拓扑序，即拓扑序是否唯一。
思考最简单的有差异拓扑序：只有2个位置不同，且相邻。
枚举每个位置的最后一次操作 $pre$，如果前一步操作也改变该位置，那么无法交换，否则可以。
```c++
for (int i = 1; i <= n; i ++ ){
        int p;
        cin >> p;
        while (p -- ){
            int x; cin >> x;
            pre[x] = i;
            edge[i].insert(x);
        }
    }
    //只要不是链，就行
    for (int i = 1; i <= m; i ++ )
        if (pre[i])
            if (edge[pre[i] - 1].count(i)) flag[pre[i]] = true;
    int px = -1; //交换 px - 1 和 px
    for (int i = 2; i <= n; i ++ )
        if (!flag[i]){
            px = i;  break;
        }
```
# C. Primitive Root #
$g \oplus (P - 1) \equiv 1(mod P) \implies g \oplus (P - 1) = kP + 1 (k是非负整数)$ 
解法一：
> [!TIP]
> $a + b <= a \oplus b <= a + b$

解法二：
$[0, m]内的数异或一个固定的数x会划分为logm段区间，固定前缀，计算 [L, R]区间有多少个形如 KP + 1的数$
```c++
ll p, m;
 
ll get(ll x)
{
    -- x;
    if (x < 0)
        return 0;
    return x / p + 1; // 正整数(下取整) + 整数0
}
 
ll query(ll L, ll R)
{
    return get(R) - get(L - 1);
}
 
void solve()
{
    cin >> p >> m;
    ll ans = 0;
    for (int i = 60; i >= 0; i--)
        if (m >> i & 1){
            //无与伦比的写法
            ll L = m >> i ^ 1;
            L ^= (p - 1) >> i;
            L <<= i;
            ans += query(L, L + (1ll << i) - 1);
        }
    ans += query(m ^ (p - 1), m ^ (p - 1));
    cout << ans << '\n';
}
```