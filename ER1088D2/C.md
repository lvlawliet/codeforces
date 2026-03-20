## 题目翻译

**泉水**

Alice、Bob、Carol 三人去泉边取水：
- Alice 每隔 a 天来一次（第 a, 2a, 3a, … 天）
- Bob 每隔 b 天来一次（第 b, 2b, 3b, … 天）
- Carol 每隔 c 天来一次（第 c, 2c, 3c, … 天）

规则：
- 只有 1 人来：取 **6 升**
- 有 2 人来：各取 **3 升**
- 有 3 人来：各取 **2 升**

求 m 天内三人各取了多少升水。

---

## 解题思路

### 关键问题

m 最大为 10¹⁷，不能枚举每一天，需要**数学方法**。

### 核心思路：容斥原理

对于 Alice，她取水的天数分三种情况：

| 情况 | 每次取水量 | 天数计算 |
|------|-----------|---------|
| 只有 Alice 来 | 6 升 | 只是 a 的倍数，不是 b 或 c 的倍数 |
| Alice 和另一人同来 | 3 升 | a 和 b 的公倍数，或 a 和 c 的公倍数（但不含三人同来） |
| 三人同来 | 2 升 | a、b、c 的公倍数 |

所以 Alice 的总水量：

$$W_A = 6 \cdot N_A^{only} + 3 \cdot N_{AB}^{only} + 3 \cdot N_{AC}^{only} + 2 \cdot N_{ABC}$$

用辅助函数 `f(x)` = m 天内 x 的倍数个数 = `⌊m/x⌋`，则：

- $N_{ABC}$ = f(lcm(a,b,c))
- $N_{AB}$ = f(lcm(a,b))， $N_{AB}^{only}$ = $N_{AB}$ - $N_{ABC}$（同来但不含三人）
- $N_{AC}$ = f(lcm(a,c))， $N_{AC}^{only}$ = $N_{AC}$ - $N_{ABC}$
- $N_A^{only}$ = f(a) - $N_{AB}$ - $N_{AC}$ + $N_{ABC}$（容斥）

代入整理：

$$W_A = 6 \cdot f(a) - 3 \cdot f(\text{lcm}(a,b)) - 3 \cdot f(\text{lcm}(a,c)) + 2 \cdot f(\text{lcm}(a,b,c))$$

> **注意**：lcm 可能超过 10¹⁷，此时 f = 0，需要处理溢出。

---

## C++ 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef __int128 lll;

ll safe_lcm(ll x, ll y)
{
    ll g = __gcd(x, y);
    lll val = (lll)x / g * y;
    if (val > (lll)2e17)
        return (ll)2e17;
    return (ll)val;
}

ll f(ll x, ll m)
{
    if (x > m)
        return 0;
    return m / x;
}

ll calc(ll self, ll other1, ll other2, ll m)
{
    ll ab = safe_lcm(self, other1);
    ll ac = safe_lcm(self, other2);
    ll abc = safe_lcm(ab, other2);

    return 6 * f(self, m) - 3 * f(ab, m) - 3 * f(ac, m) + 2 * f(abc, m);
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int t;
    cin >> t;
    while (t--)
    {
        ll a, b, c, m;
        cin >> a >> b >> c >> m;

        cout << calc(a, b, c, m) << " "
             << calc(b, a, c, m) << " "
             << calc(c, a, b, m) << "\n";
    }
    return 0;
}
```

---

## 公式推导总结

对于 Alice（另外两人周期为 b, c），设：

$$W_A = 6\lfloor\frac{m}{a}\rfloor - 3\lfloor\frac{m}{\text{lcm}(a,b)}\rfloor - 3\lfloor\frac{m}{\text{lcm}(a,c)}\rfloor + 2\lfloor\frac{m}{\text{lcm}(a,b,c)}\rfloor$$

Bob、Carol 对称地套用同一公式即可。

