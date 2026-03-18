# Stamina and Tasks

## 1. 题目翻译 (Translation)

### 题目描述
你有 $n$ 个任务需要处理。每个任务 $i$ 有一个价值 $c_i$ 和一个难度 $p_i$。你的初始耐力（Stamina）为 $S = 1$。
你需要按从 $1$ 到 $n$ 的顺序依次处理这些任务。对于每个任务，你有两个选择：
1.  **放弃任务**：什么都不会发生。
2.  **完成任务**：你将获得 $S \cdot c_i$ 分。完成任务后，你的耐力 $S$ 会降为 $S \cdot (1 - \frac{p_i}{100})$。

你的目标是在处理完所有任务后，使总得分最大化。

### 输入格式
* 第一行包含测试用例数量 $t$ ($1 \le t \le 10^3$)。
* 每个测试用例的第一行是一个整数 $n$ ($1 \le n \le 10^5$)。
* 接下来的 $n$ 行每行包含两个整数 $c_i$ 和 $p_i$ ($1 \le c_i \le 100, 0 \le p_i \le 100$)。
* 保证所有测试用例的 $n$ 之和不超过 $10^5$。

### 输出格式
* 对于每个测试用例，输出一个实数，代表可能获得的最大分数。如果误差不超过 $10^{-6}$，则认为答案正确。

---

### 2. 解题思路 (Approach)

为了简化逻辑，我们采用 **逆向动态规划 (Backward DP)**。由于耐力 $S$ 对后续分数的影响是乘法关系的，从后往前推导可以避免维护复杂的累乘状态。

**状态定义：**
定义 $dp[i]$ 为：假设在开始处理第 $i$ 个任务时，耐力 $S = 1$ ，从任务 $i$ 到任务 $n$ 所能获得的最大额外分数。

当面对第 $i$ 个任务时，有两种决策：

1.  **不接任务 $i$** ：
    * 耐力保持为 $1$ 。
    * 当前收益为 $0$ ，后续收益为 $dp[i+1]$ 。
    * 总分数： $dp[i+1]$ 。
2.  **接任务 $i$** ：
    * 立即获得分数： $1 \cdot c_i$ 。
    * 完成任务后，耐力缩放因子变为： $(1 - \frac{p_i}{100})$ 。
    * 因为耐力是线性缩放的，后续任务在当前任务完成后的最大收益也会等比例缩放。
    * 总分数： $c_i + (1 - \frac{p_i}{100}) \cdot dp[i+1]$ 。

#### 状态转移方程 (State Transition Equation)

$$dp[i] = \max \left( dp[i+1], \ c_i + \left( 1 - \frac{p_i}{100} \right) \cdot dp[i+1] \right)$$



* **基准情况 (Base Case)** ： $dp[n+1] = 0$ （没有更多任务，得分为 $0$ ）。
* **最终答案** ： $dp[1]$ （即初始耐力为 $1$ 时，从第 $1$ 个任务开始的最大得分）。

#### 复杂度分析 (Complexity)

* **时间复杂度** ： $O(n)$ 。只需从 $n$ 到 $1$ 遍历一次任务列表。
* **空间复杂度** ： $O(1)$ 。在代码实现中，我们可以只用一个变量记录 `current_max_points` ，不需要开辟完整的 $dp$ 数组。

---

## 3. C++ 代码实现 (Implementation)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <iomanip>

using namespace std;

void solve()
{
    int n;
    if (!(cin >> n))
        return;

    struct Task
    {
        int c;
        double r;
    };
    vector<Task> tasks(n);
    for (int i = 0; i < n; ++i)
    {
        int c_val, p_val;
        cin >> c_val >> p_val;
        tasks[i].c = c_val;
        tasks[i].r = 1.0 - (p_val / 100.0);
    }

    double current_max_points = 0.0;

    for (int i = n - 1; i >= 0; --i)
    {
        double skip = current_max_points;
        double take = (double)tasks[i].c + tasks[i].r * current_max_points;

        current_max_points = max(skip, take);
    }

    cout << fixed << setprecision(10) << current_max_points << endl;
}

int main()
{
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int t;
    if (!(cin >> t))
        return 0;
    while (t--)
    {
        solve();
    }
    return 0;
}
```
