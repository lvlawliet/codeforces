## 题目翻译

**分数之和**

定义分数 x/y 的**一次增加操作**为：
- 分子 x 加 1，或
- 若分母 y > 1，分母 y 减 1（不化简）

给定数组 b 和整数 k，从数组 `1/b₁, 1/b₂, …, 1/bₘ` 出发，恰好执行 k 次增加操作，使分数之和最大，记为 MSF(b, k)。

给定数组 a₁…aₙ 和 k₁…kₘ，对每个 kᵢ，计算所有子数组的 MSF 之和（模 998244353）。

---

## 解题思路

### Step 1：单个子数组的 MSF 公式

对子数组 [l,r]，设 S = ∑aᵢ，len = r-l+1，最优策略是将所有分母减到同一个值 v，再用剩余操作加分子。

最优 v 满足：
$$v = \min\left(a[l..r],\ \left\lceil\frac{S-k}{len}\right\rceil\right)$$

当 k 足够大（k ≥ S - len）时直接退化，否则：

$$\boxed{\text{MSF}(l,r,k) = len - \frac{S-k}{v}}$$

---

### Step 2：v 就是子数组最小值

**关键观察**：当 k 不太大时，最优 v 恰好等于 `min(a[l..r])`。

具体地，以 v = min 为最优的条件是：
$$k < S - len \cdot (v - 1) \iff k < \underbrace{(a_{\min} - 1)}_{\text{thresh}}$$

即当 `k < thresh = a[min] - 1` 时，v = min(a[l..r])。

---

### Step 3：单调栈枚举最小值

用单调栈，对每个位置 i，求：
- `prv[i]`：左边第一个**严格小于** a[i] 的位置
- `nxt[i]`：右边第一个**小于等于** a[i] 的位置

则 a[i] 作为子数组最小值的子数组数为：
$$\text{count}(i) = (i - \text{prv}[i]) \times (\text{nxt}[i] - i)$$

---

### Step 4：答案拆成三项

对所有子数组求和，将答案拆为三部分：

$$\text{Answer}(k) = \underbrace{S_1}_{\text{固定项}} + \underbrace{k \cdot AA}_{\text{线性项}} + \underbrace{sp \cdot k - sq}_{\text{离线修正项}}$$

**固定项 S1**：k=0 时的基础贡献

$$S_1 = \sum_i \frac{1}{a[i]} \cdot (\text{位置 i 对所有以 i 为最小值的子数组的元素和之和})$$

位置 j 对以 i 为最小值的子数组中的贡献次数 = i 左边可选端点数 × i 右边可选端点数，因此：

$$S_1 = \sum_i \frac{i \cdot (n - i + 1)}{a[i]} \pmod{p}$$

**线性项 AA**：每个子数组贡献 k/v = k/a[i]

$$AA = \sum_i \frac{\text{count}(i)}{a[i]}$$

**离线修正项**：当 k ≥ thresh = a[i]-1 时，该子数组的 v 不再是 a[i]，需要修正。

每个位置 i 产生一个事件：
$$\text{thresh} = a[i] - 1, \quad p = \text{count}(i) \cdot \left(1 - \frac{1}{a[i]}\right), \quad r = p \cdot \text{thresh}$$

当 k 超过 thresh 后，修正累加量为 `p·k - r`。

---

### Step 5：离线处理查询

k 数组**非递减**，按阈值排序所有事件，双指针扫描：

```
将所有事件按 thresh 排序
对每个查询 k：
    将所有 thresh ≤ k 的事件加入：sp += p，sq += r
    答案 = S1 + k·AA + sp·k - sq
```

---

## C++ 代码

```cpp
#include <iostream>
#include <vector>
#include <stack>
#include <algorithm>
#include <tuple>

using namespace std;

typedef long long ll;
const int MOD = 998244353;

ll modpow(ll b, ll e) 
{
    ll ans = 1;
    b %= MOD;
    for (; e; e /= 2) 
    {
        if (e & 1) ans = ans * b % MOD;
        b = b * b % MOD;
    }
    return ans;
}

ll modinv(ll x) 
{
    return modpow(x, MOD - 2);
}

int main() 
{
    ios::sync_with_stdio(false);
    cin.tie(0);

    int n, m;
    if (!(cin >> n >> m)) return 0;

    vector<ll> a(n + 2);
    for (int i = 1; i <= n; i++) cin >> a[i];

    vector<ll> ks(m);
    for (auto& x : ks) cin >> x;

    vector<int> prv(n + 2, 0);
    vector<int> st;
    for (int i = 1; i <= n; i++) 
    {
        while (!st.empty() && a[st.back()] > a[i]) st.pop_back();
        prv[i] = st.empty() ? 0 : st.back();
        st.push_back(i);
    }
    
    vector<int> nxt(n + 2, n + 1);
    st.clear();
    for (int i = n; i >= 1; i--) 
    {
        while (!st.empty() && a[st.back()] >= a[i]) st.pop_back();
        nxt[i] = st.empty() ? n + 1 : st.back();
        st.push_back(i);
    }

    ll S1 = 0;
    ll AA = 0;
    vector<tuple<ll, ll, ll>> its;

    for (int i = 1; i <= n; i++) 
    {
        ll nl = i - prv[i];
        ll nr = nxt[i] - i;
        ll count = nl * nr % MOD;
        
        ll inva = modinv(a[i]);
        ll sc = (ll)i % MOD * ((n - i + 1) % MOD) % MOD;
        S1 = (S1 + inva * sc) % MOD;

        AA = (AA + count * inva) % MOD;

        ll thresh = a[i] - 1;
        ll p = count * (1 - inva + MOD) % MOD;
        ll r = p * (thresh % MOD) % MOD; 
        its.emplace_back(thresh, p, r);
    }

    sort(its.begin(), its.end());

    ll sp = 0, sq = 0;
    int pt = -1;
    for (int qi = 0; qi < m; qi++) 
    {
        ll k = ks[qi];
    
        while (pt + 1 < n && get<0>(its[pt + 1]) <= k) 
        {
            ++pt;
            sp = (sp + get<1>(its[pt])) % MOD;
            sq = (sq + get<2>(its[pt])) % MOD;
        }

        ll fi = (k % MOD * AA % MOD);

        ll se = (sp * (k % MOD) % MOD - sq + MOD) % MOD;
        
        cout << (S1 + fi + se) % MOD << '\n';
    }

    return 0;
}
```
