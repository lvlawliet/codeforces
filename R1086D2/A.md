# Bingo Candies - 题目解析与实现

## 1. 题目翻译 (Translation)

### 题目描述
爱丽丝有一个魔法板，是一个 $n \times n$ 的网格；每个格子里都有一颗有颜色的糖果。第 $i$ 行第 $j$ 列的糖果颜色为 $a_{i,j}$ 。

鲍勃想知道他是否可以 **重新排列** 棋盘上的糖果，使得：
* **没有任何一行** 包含 $n$ 颗颜色相同的糖果（即不是单色行）。
* **没有任何一列** 包含 $n$ 颗颜色相同的糖果（即不是单色列）。

你的任务是确定是否存在这样的重新排列方案。

### 输入格式
* 第一行包含测试用例数量 $t$ ($1 \le t \le 500$)。
* 每个测试用例第一行是一个整数 $n$ ($1 \le n \le 100$)。
* 接下来的 $n$ 行每行包含 $n$ 个整数，代表糖果的颜色 $a_{i,j}$ ($1 \le a_{i,j} \le n^2$)。

---

## 2. 解题思路 (Approach)

### 核心逻辑分析
这是一个关于 **鸽巢原理** 的构造约束问题。

1.  **约束分析**：要打破某一行或某一列的单色状态，该行/列中必须至少包含一颗 “异色” 糖果。
2.  **极端情况**：假设某种颜色 $C$ 出现了 $cnt_C$ 次。
    * 为了让每一行都不是纯色 $C$ ，我们至少需要 $n$ 个 **非 $C$ 颜色** 的糖果，将它们分别放在 $n$ 行中。
    * 为了让每一列也不是纯色 $C$ ，我们同样需要 $n$ 个 **非 $C$ 颜色** 的糖果。
3.  **对角线策略**：如果我们拥有至少 $n$ 个非 $C$ 色的糖果，我们可以将这些异色糖果放置在矩阵的 **主对角线** 上（即坐标 $(i, i)$ 处）。
    * 这样，每一行都包含了一个异色点，每一列也包含了一个异色点。
    * 因此，只要剩下的 “杂色” 糖果数量足够填满对角线，颜色 $C$ 就无法统治任何一行或一列。

### 判定公式
对于网格中出现的任意一种颜色 $C$ ，设其出现次数为 $cnt_C$ ：

* **成立条件** ： $cnt_C \le n^2 - n$
* **失效条件** ： $cnt_C > n^2 - n$ （这意味着其他颜色的糖果总数少于 $n$ 颗，不足以破坏掉所有的 $n$ 条行约束或列约束）。

> **注意** ：当 $n = 1$ 时， $n^2 - n = 0$ 。由于格子里必然有一种颜色，其 $cnt_C = 1$ ，因为 $1 > 0$ ，所以 $n = 1$ 时结果始终为 **NO** 。

---

## 3. C++ 代码实现 (Implementation)

``` cpp
#include <iostream>
#include <vector>
#include <unordered_map>
#include <algorithm>

using namespace std;

void solve()
{
    int n;
    if (!(cin >> n))
        return;

    unordered_map<int, int> color_counts;
    int total_cells = n * n;

    for (int i = 0; i < total_cells; ++i)
    {
        int color;
        cin >> color;
        color_counts[color]++;
    }

    int limit = n * n - n;
    bool possible = true;

    for (auto const &[color, count] : color_counts)
    {
        if (count > limit)
        {
            possible = false;
            break;
        }
    }

    if (possible)
    {
        cout << "YES" << endl;
    }
    else
    {
        cout << "NO" << endl;
    }
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
