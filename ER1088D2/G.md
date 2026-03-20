## 题目翻译

**网格路径**

给定 n 行 m 列的网格，芯片从 (1,1) 出发，每次可以向左、向右或向下移动，不能离开网格。

定义路径为芯片**至少访问一次的格子的集合**（与顺序无关）。

求所有可能的路径数，模 mod。

---

## 解题思路

### Step 1：前缀差分转化

定义 `f(n, c)` = 从 (1,1) 出发，走 n 行，且路径中**最右列 ≤ c** 的路径集合数。

则答案为：
$$\text{ans} = f(N, M) - f(N, M-1)$$

将"最右列恰好能用到第 M 列"转化为前缀相减，极大简化状态。

---

### Step 2：f(n, c) 的递推公式

每行被访问的格子是连续区间，设第 n 行区间右端点为 c，左端点 l 可取 1 到 c，共 c 种选择。对第 n+1 行右端点为 c' 的情况，需要与第 n 行有交叉。

经整理，`f(n+1, c)` 满足：

$$f(n+1,\ c) = \frac{c(c+1)}{2} \cdot [f(n,M) - f(n,M-1)] - \sum_{j=0}^{c-1}(c-j)\cdot[f(n,j)-f(n,j-1)] - \sum_{r=1}^{c} r\cdot[f(n,M-r)-f(n,M-r-1)]$$

直观理解：
- 第一项：第 n+1 行右端点为 c，左端点任取 1..c，共 c(c+1)/2 种区间，每种都能接在"最右恰好为 M"的行后面
- 第二项：减去左端点超出范围的重复计数
- 第三项：减去右端点不够的情况

---

### Step 3：状态向量设计

维护长度 `M+2` 的向量：

| 下标 | 含义 |
|------|------|
| `v[j]`（j=0..M-1） | `f(n, j)` |
| `v[M]` | `f(n, M)`（答案的被减数）|
| `v[M+1]` | 常数 1（锚点，处理递推中的常数项）|

---

### Step 4：构造转移矩阵

对每个右端点 k（1 ≤ k ≤ M），`trans[k][...]` 的系数为：

```
trans[k][M]   += k*(k+1)/2      // f(n,M) 的正贡献
trans[k][M+1] += k*(k+1)/2      // 常数锚点对齐

trans[k][j]   -= (k-j)          // j = 0..k-1，减去左侧重复
trans[k][M-r] -= r              // r = 1..k，减去右侧不足
```

常数项自循环：`trans[M+1][M+1] = 1`

矩阵大小仅为 **(M+2) × (M+2)**，相比原始 O(m²) 维状态压缩巨大。

---

### Step 5：矩阵快速幂 + 提取答案

初始向量：`init[M+1] = 1`（从常数锚点出发，代表第 0 行的初始状态）

对转移矩阵做 N 次幂，作用于初始向量后：

$$\text{ans} = \text{finalv}[M] - \text{finalv}[M-1] \pmod{mod}$$

---

## C++ 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

int main()
{
    ll N, MOD;
    int M;
    cin >> N >> M >> MOD;
    if (N == 0)
    {
        cout << 0 << '\n';
        return 0;
    }
    if (M == 0)
    {
        cout << 0 << '\n';
        return 0;
    }

    int D = M + 2;
    vector<vector<ll>> trans(D, vector<ll>(D, 0LL));

    for (int k = 1; k <= M; ++k)
    {
        ll num = (ll)k * (k + 1) / 2 % MOD;
        trans[k][M] = (trans[k][M] + num) % MOD;
        trans[k][M + 1] = (trans[k][M + 1] + num) % MOD;

        for (int j = 0; j < k; ++j)
        {
            ll coef = (ll)(k - j) % MOD;
            trans[k][j] = (trans[k][j] - coef + MOD) % MOD;
        }
        for (int r = 1; r <= k; ++r)
        {
            int j = M - r;
            if (j >= 0)
            {
                ll coef = (ll)r % MOD;
                trans[k][j] = (trans[k][j] - coef + MOD) % MOD;
            }
        }
    }
    trans[M + 1][M + 1] = 1;

    auto mul = [&](const vector<vector<ll>> &A, const vector<vector<ll>> &B)
    {
        vector<vector<ll>> C(D, vector<ll>(D, 0LL));
        for (int i = 0; i < D; ++i)
            for (int k = 0; k < D; ++k)
                if (A[i][k])
                {
                    for (int j = 0; j < D; ++j)
                    {
                        C[i][j] = (C[i][j] + A[i][k] * B[k][j] % MOD) % MOD;
                    }
                }
        return C;
    };

    auto powmat = [&](vector<vector<ll>> base, ll exp)
    {
        vector<vector<ll>> res(D, vector<ll>(D, 0LL));
        for (int i = 0; i < D; ++i)
            res[i][i] = 1;
        while (exp > 0)
        {
            if (exp & 1)
                res = mul(res, base);
            base = mul(base, base);
            exp >>= 1;
        }
        return res;
    };

    auto powered = powmat(trans, N);

    vector<ll> init(D, 0LL);
    init[M + 1] = 1;

    vector<ll> finalv(D, 0LL);
    for (int i = 0; i < D; ++i)
    {
        for (int j = 0; j < D; ++j)
        {
            finalv[i] = (finalv[i] + powered[i][j] * init[j] % MOD) % MOD;
        }
    }

    ll ans = (finalv[M] - finalv[M - 1] + MOD) % MOD;
    cout << ans << '\n';
    return 0;
}
```
