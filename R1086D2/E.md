# Counting Cute Arrays — 解题思路

## 中文翻译

**题目名称：计数可爱数组**

给定一个长度为 $n$ 的正整数数组 $A = [A_1, A_2, \dots, A_n]$。定义数组 $f(A)$ 如下：
对于每个 $i \in [1, n]$：
* 如果存在 $j < i$ 且 $A_j < A_i$，则 $f(A)_i$ 是满足条件的最大的 $j$（即 $i$ 左侧最近的小于 $A_i$ 的元素的下标）。
* 否则， $f(A)_i = 0$。

如果存在某个数组 $A$ 使得 $f(A) = P$，则称非负整数数组 $P$ 为**可爱数组**。

现在给定一个长度为 $n$ 的数组 $X$，其中部分元素为 $-1$。你需要计算：将所有的 $-1$ 替换为 $0 \dots n$ 中的整数后，能够形成的“可爱数组”的数量。
结果对 $998244353$ 取模。

---

## 第一步：理解 cute array 的结构

### cute array 即 Cartesian Tree 父指针

$f(A)_i = k$ 的含义是： $A_k < A_i$，且对所有 $k < j < i$ 都有 $A_j \geq A_i$。

这正好是**单调递增栈**（从左到右处理 $A$ 时维护的栈）的父指针结构——每个元素的"父节点"是栈中它正下方的元素。

因此， $P$ 是 cute array 当且仅当 $P$ 是某个序列 $A$ 的 **Cartesian Tree 父指针数组**。

### 充要条件

$P$ 是 cute array 当且仅当：

1. $P_1 = 0$（第一个元素无左邻，父节点为虚拟根 $0$）
2. $P_i < i$（父节点下标严格小于自身）
3. 对所有 $i$，若 $P_i = k > 0$，则对所有 $k < j < i$，有 $P_j \geq k$

**条件 3 的含义**：若节点 $i$ 的父节点是 $k$，则区间 $(k, i)$ 内的所有节点，其父节点下标都必须 $\geq k$。直觉上，这保证了 Cartesian Tree 的"右子树包含性"——区间 $(k,i)$ 内的节点不能"跨越"$k$ 向左指。

> **示例**： $[0,0,0,1]$ 非法—— $P_4=1$ 要求 $P_2, P_3 \geq 1$，但 $P_2=P_3=0 < 1$，矛盾。

---

## 第二步：预处理——有效性检查

在枚举之前，先对输入的固定值做一轮合法性检查。

对每个 $X_i \neq -1$（设 $X_i = k$）：

- 若 $k \geq i$：非法（父节点下标必须 $< i$）
- 若存在 $j > i$ 使得 $X_j \neq -1$ 且 $k < X_j < i$：非法

第二条检查的含义：若 $P_i = k$ 固定，则区间 $(k, i)$ 内不能有固定值落在 $(k, i)$ 之外（即 $X_j$ 本身在 $(k, i)$ 内意味着 $P_j$ 指向了比 $k$ 更右的地方，而 $j > i$ 说明这个父指针"跨越"了 $i$，破坏了 Cartesian Tree 的结构）。

```cpp
for (int i = 1; i <= n; i++)
{
    if (a[i] == -1)
        continue;
    if (a[i] >= i)
    {
        cout << 0 << "\n";
        return;
    }
    for (int j = i + 1; j <= n; j++)
        if (a[j] != -1 && a[i] < a[j] && a[j] < i)
        {
            cout << 0 << "\n";
            return;
        }
}
```

---

## 第三步：Cartesian Tree 的递归子结构

### 核心分解

考虑整个序列 $P[1..n]$ 对应的 Cartesian Tree，以虚拟节点 $0$ 为根。

**关键观察**： $P_n = k$ 意味着节点 $n$ 的父节点是 $k$。这将整个问题分解为两个独立子问题：

- **左子问题**：位置 $1..k$ 构成 $k$ 的左子树部分，满足 $X[1..k]$ 的约束
- **右子问题**：位置 $k+1..n-1$ 构成以 $k$ 为根的右子树，满足 $X[k+1..n-1]$ 的约束

两个子问题完全独立，总方案数 = 左方案数 × 右方案数。

### 推广到 `solve(l, r)`

定义 `solve(l, r)` 处理区间 $(l, r)$ 内的节点，其中隐含条件是「节点 $r$ 的父节点为 $l$」。

具体地，`solve(l, r)` 统计满足 $X$ 约束的、所有 $P[l+1..r-1]$ 的合法填法数（其中 $P[r] = l$ 已固定）。

初始调用为 `solve(0, n+1)`，其中 $n+1$ 是虚拟右端点， $0$ 是虚拟根。

---

## 第四步：固定节点的子树合并

在 `solve(l, r)` 内，**从右往左**扫描区间 $(l, r)$ 内的节点：

```
遍历 i = r-1, r-2, ..., l+1：
  若 a[i] != -1（固定节点）：
    递归调用 solve(a[i], i)，得到子树方案数
    res *= 子树方案数
    将这整棵子树"合并"为序列 b 中的一个标记为 1 的元素
    跳过子树内部（i 跳到 a[i]）
  若 a[i] == -1（自由节点）：
    在序列 b 中追加标记 -1
```

将 $b$ 翻转得到从左到右的顺序。

**为什么固定子树能合并为单个元素？**

当 $a[i] \neq -1$ 时，节点 $i$ 的父节点 $a[i]$ 已固定，这棵子树内部的结构与外部完全独立（已由递归计算），对外部的单调栈而言，整棵子树的效果等同于"压入了一个新元素"——它只会使栈深度 $+1$，不会引发任何弹栈。

---

## 第五步：对序列 b 做 DP

处理完固定子树后，`b[]` 是一个只含 `1` 和 `-1` 的序列：

- `b[i] = 1`：固定节点（已合并的子树），等价于 **push** 操作
- `b[i] = -1`：自由节点，可以 push 或弹出到任意深度

### DP 定义

$$dp[\text{step}][j] = \text{处理完 } b[1..\text{step}] \text{ 后，单调栈大小为 } j \text{ 的方案数}$$

**初始状态**： $dp[0][1] = 1$（区间起点 $l$ 本身在栈中，栈大小为 $1$）

### DP 转移

**情形一： $b[\text{step}] = 1$（固定节点，push）**

固定节点只能 push，栈大小严格 $+1$：

$$dp[\text{step}][j] = dp[\text{step}-1][j-1]$$

**情形二： $b[\text{step}] = -1$（自由节点）**

自由节点可以：
- **Push**：不弹出任何元素，直接压入，栈大小变为旧大小 $+1$
- **弹出 $t$ 个再 push**：栈大小从 $j'$ 变为 $j' - t + 1 = j$，即 $j' \geq j - 1$

因此，新栈大小 $j$ 对应旧栈大小 $j' \in [j-1, \text{step}]$：

$$dp[\text{step}][j] = \sum_{j'=j-1}^{\text{step}} dp[\text{step}-1][j']$$

利用前缀和数组 $s[k] = \sum_{j=1}^{k} dp[\text{step}-1][j]$，可以 $O(1)$ 计算：

$$dp[\text{step}][j] = s[\text{step}] - s[\max(j-2, 0)]$$

### 最终结果

`solve(l, r)` 的返回值为：

$$\text{res} \times \sum_{j=1}^{m+1} dp[m][j] = \text{res} \times s[m+1]$$

其中 $\text{res}$ 是所有固定子树递归结果的乘积， $s[m+1]$ 是本层自由序列的方案数。

---

## 完整示例

以 $n=4$， $X = [-1, -1, 1, -1]$（期望答案 $3$）为例。

**有效性检查**： $X_3 = 1 < 3$，无违规。

**`solve(0, 5)`**，从右往左扫描 $i=4,3,2,1$：
- $i=4$： $a[4]=-1$， $b$ 追加 $-1$
- $i=3$： $a[3]=1 \neq -1$，递归 `solve(1, 3)`， $b$ 追加 $1$， $i$ 跳到 $1$
- $i=1$： $a[1]=-1$， $b$ 追加 $-1$

$b = [-1, -1, 1]$ 翻转后 $b = [-1, 1, -1]$。

**`solve(1, 3)`**，扫描 $i=2$：
- $i=2$： $a[2]=-1$， $b'$ 追加 $-1$

$b' = [-1]$， $dp'[0][1]=1$， $dp'[1][2] = s'[1]-s'[0]=1$，返回 $s'[2] = 1$。

**回到 `solve(0, 5)`**， $\text{res} = 1$，对 $b=[-1,1,-1]$ 做 DP：

| step | b[step] | dp[step][2] | dp[step][3] | dp[step][4] | s[2] | s[3] | s[4] |
|------|---------|-------------|-------------|-------------|------|------|------|
| 0    | —       | 0           | 0           | 0           | 1    | 1    | 1    |
| 1    | $-1$    | 1           | 0           | 0           | 1    | 1    | 1    |
| 2    | $1$     | 0           | 1           | 0           | 0    | 1    | 1    |
| 3    | $-1$    | 1           | 1           | 1           | 1    | 2    | 3    |

答案 $= s[4] = 3$。✓

---

## 完整代码

```cpp
#pragma GCC optimize("Ofast")
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int maxn = 5010, mod = 998244353;

int n;
int a[maxn];
int dp[maxn][maxn], s[maxn];

int solve(int l, int r)
{
    int res = 1;
    vector<int> b;

    for (int i = r - 1; i > l; i--)
    {
        if (a[i] != -1)
        {
            res = res * solve(a[i], i) % mod;
            b.push_back(1);
            i = a[i] + 1;
        }
        else
        {
            b.push_back(-1);
        }
    }

    reverse(b.begin(), b.end());
    int m = b.size();

    for (int i = 0; i <= m + 1; i++)
    {
        s[i] = 0;
        for (int j = 0; j <= m + 1; j++)
            dp[i][j] = 0;
    }

    dp[0][1] = s[1] = 1;

    for (int step = 1; step <= m; step++)
    {
        for (int j = 2; j <= step + 1; j++)
        {
            if (b[step - 1] == -1)
                dp[step][j] = (s[step] - s[max(j - 2, 0LL)] + mod) % mod;
            else
                dp[step][j] = dp[step - 1][j - 1];
        }
        for (int j = 1; j <= step + 1; j++)
            s[j] = (s[j - 1] + dp[step][j]) % mod;
    }

    return res * s[m + 1] % mod;
}

void solve_case()
{
    cin >> n;
    for (int i = 1; i <= n; i++)
        cin >> a[i];

    for (int i = 1; i <= n; i++)
    {
        if (a[i] == -1)
            continue;
        if (a[i] >= i)
        {
            cout << 0 << "\n";
            return;
        }
        for (int j = i + 1; j <= n; j++)
            if (a[j] != -1 && a[i] < a[j] && a[j] < i)
            {
                cout << 0 << "\n";
                return;
            }
    }

    cout << solve(0, n + 1) << "\n";
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    int T;
    cin >> T;
    while (T--)
        solve_case();
    return 0;
}
```
