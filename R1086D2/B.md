# Cyclists

## 1. 题目翻译 (Translation)

### 题目背景
鲍勃在玩一款塔防手游。牌组中有 $n$ 张卡牌排成一个队列。在任何时刻，鲍勃只能从队列的 **前 $k$ 个位置** 中选择一张卡牌打出。

### 规则说明
1.  **打牌机制**：从前 $k$ 张卡中选一张打出后，该卡会被移出队列，然后重新放入队列的 **末尾**。原先排在它后面的所有卡牌都会向前移动一位。
2.  **核心目标**：队列中有一张特殊的卡牌叫做“获胜卡”，初始位置在第 $p$ 位。鲍勃希望在总能量消耗不超过 $m$ 的前提下，尽可能多地打出这张“获胜卡”。
3.  **能量消耗**：初始第 $i$ 位的卡牌每次被打出都会消耗 $a_i$ 点能量。

### 任务
计算在能量限制 $m$ 内，获胜卡最多能被打出多少次。

---

## 2. 解题思路 (Approach)

为了最大化获胜卡的打出次数，我们要用 **最小的代价** 来移动卡牌。

### 核心观察
* **获胜卡的位置移动**：
    * 如果获胜卡在位置 $pos > k$，它是不可用的。
    * 每打出一张处于获胜卡 **前面** 的卡（即位置 $i < pos$ 且 $i \le k$），获胜卡的位置就会减 1。
* **首次打出的代价 ($E_1$)**：
    * 如果初始位置 $p \le k$，可以直接打出获胜卡，代价为 $a_p$。
    * 如果 $p > k$，我们需要先打出 $(p - k)$ 张排在获胜卡前面的卡，让它进入前 $k$ 范围。为了节省能量，我们应该从初始前 $p-1$ 张卡中，挑选 **代价最小的 $(p - k)$ 张** 打出。
* **随后每次打出的代价 ($E_{next}$)**：
    * 打出获胜卡后，它会移动到队列末尾（位置 $n$）。
    * 要再次打出它，如果 $n > k$，需要打出 $(n - k)$ 张其他卡牌将其顶回前 $k$ 位。
    * 此时我们可以从除了获胜卡之外的所有 $(n - 1)$ 张卡中，挑选 **代价最小的 $(n - k)$ 张** 打出。

### 算法步骤
1.  记获胜卡代价为 $cost_{win} = a_p$。
2.  **计算第一次代价**：
    * 若 $p \le k$, $E_1 = cost_{win}$。
    * 若 $p > k$, $E_1 = (\text{前 } p-1 \text{ 张卡中最小的 } p-k \text{ 个代价之和}) + cost_{win}$。
3.  **计算后续重复代价**：
    * 若 $n \le k$, $E_{next} = cost_{win}$。
    * 若 $n > k$, $E_{next} = (\text{所有其他 } n-1 \text{ 张卡中最小的 } n-k \text{ 个代价之和}) + cost_{win}$。
4.  **计算结果**：
    * 如果 $m < E_1$，结果为 $0$。
    * 否则，结果为 $1 + \lfloor \frac{m - E_1}{E_{next}} \rfloor$。

---

## 3. C++ 代码实现 (Implementation)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

typedef long long ll;

void solve()
{
    int n, k, p;
    ll m;
    cin >> n >> k >> p >> m;

    vector<ll> a(n);
    for (int i = 0; i < n; ++i)
    {
        cin >> a[i];
    }

    ll win_cost = a[p - 1];

    ll e1 = win_cost;
    if (p > k)
    {
        vector<ll> prefix_others;
        for (int i = 0; i < p - 1; ++i)
        {
            prefix_others.push_back(a[i]);
        }
        sort(prefix_others.begin(), prefix_others.end());

        int needed = p - k;
        for (int i = 0; i < needed; ++i)
        {
            e1 += prefix_others[i];
        }
    }

    if (m < e1)
    {
        cout << 0 << endl;
        return;
    }

    ll e_next = win_cost;
    if (n > k)
    {
        vector<ll> all_others;
        for (int i = 0; i < n; ++i)
        {
            if (i == p - 1)
                continue;
            all_others.push_back(a[i]);
        }
        sort(all_others.begin(), all_others.end());

        int needed = n - k;
        for (int i = 0; i < needed; ++i)
        {
            e_next += all_others[i];
        }
    }

    ll remaining_m = m - e1;
    ll count = 1 + (remaining_m / e_next);

    cout << count << endl;
}

int main()
{
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
    return 0;
}
```
