## 题目翻译

**数字之和（再来一次）**

对正整数 x，字符串 S(x) 的构造过程：
1. 初始字符串为空
2. 将 x 的十进制表示（无前导零）拼接到右边
3. 若 x ≤ 9，结束；否则 x 替换为各位数字之和，回到步骤 2

例如：
- S(75) = "75123"（75 → 7+5=12 → 1+2=3）
- S(30) = "303"（30 → 3+0=3）
- S(9) = "9"

给定一个数字字符串 s，重新排列其字符，使其构成某个 S(x)。保证答案存在，输出任意合法结果。

---

## 解题思路

### 核心等式推导

设 S(x) 的结构为：`[x][c₁][c₂]...[last]`，其中 c₁=digitSum(x)，c₂=digitSum(c₁)，……

对整个字符串求数字之和：

$$\text{tot} = \underbrace{\text{digitSum}(x)}_{=c_1} + \underbrace{c_1 + c_2 + \ldots + \text{last}}_{=\text{sumS}[c_1]}$$

因此：
$$\boxed{\text{tot} = y_0 + \text{sumS}[y_0]}, \quad y_0 = \text{digitSum}(x)$$

**只需枚举 y₀（即 x 的数字之和），用此等式一步验证！**

---

### 预处理

对每个 y（1 ~ 900000），预先计算 S(y) 的信息：

- `sumS[y]`：S(y) 中所有字符的数字之和
- `cntS[y]`：S(y) 中各数字字符（0~9）的出现次数

这样验证某个 y₀ 是否可行只需 **O(10)**。

---

### 完整算法

**特判**：n=1 时直接输出。

否则：

1. 统计输入字符串的各数字频次 `fr[]` 及总数字和 `tot`

2. 枚举 y₀ = 1, 2, …, tot，找第一个满足：
   - `y₀ + sumS[y₀] == tot`（等式成立）
   - `cntS[y₀][d] ≤ fr[d]`（S(y₀) 的字符能从输入中扣除）

3. 确定 y₀ 后：
   - 后缀 `tail = S(y₀)`
   - 前缀 `x` = 剩余字符降序排列（最高位取非零数字，其余从小到大填入）

4. 输出 `x + tail`

---

### 为什么枚举 y₀ 而非枚举链条？

枚举链条需要逐步构造并反复扣字符，复杂度高且容易出错。  
而 `tot = y₀ + sumS[y₀]` 这个等式将整个链条的信息**压缩成一个数**，枚举 y₀ 后一步验证即可，极为简洁。

---

## C++ 代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXY = 900010;

vector<int> sumS(MAXY);
vector<array<int, 10>> cntS(MAXY);

string getS(int v)
{
    string r = "";
    long long c = v;

    while (1)
    {
        r += to_string(c);
        if (c <= 9)
            break;
        long long n = 0, t = c;
        while (t)
        {
            n += t % 10;
            t /= 10;
        }
        c = n;
    }

    return r;
}

void pre()
{
    for (int y = 1; y < MAXY; y++)
    {
        array<int, 10> cc = {};
        int sm = 0;
        long long cur = y;

        while (1)
        {
            long long t = cur;
            while (t)
            {
                int d = t % 10;
                cc[d]++;
                sm += d;
                t /= 10;
            }
            if (cur <= 9)
                break;
            long long nx = 0;
            t = cur;
            while (t)
            {
                nx += t % 10;
                t /= 10;
            }
            cur = nx;
        }

        cntS[y] = cc;
        sumS[y] = sm;
    }
}

void solve()
{
    string s;
    cin >> s;
    int n = s.size();

    if (n == 1)
    {
        cout << s << '\n';
        return;
    }

    array<int, 10> fr = {};
    int tot = 0;
    for (char c : s)
    {
        int d = c - '0';
        fr[d]++;
        tot += d;
    }
    
    int y0 = -1;
    for (int y = 1; y <= tot && y < MAXY; y++)
    {
        if (y + sumS[y] == tot)
        {
            bool o = 1;
            for (int d = 0; d < 10; d++)
                if (cntS[y][d] > fr[d])
                    o = 0;
            if (o)
            {
                y0 = y;
                break;
            }
        }
    }

    string tl = getS(y0);
    array<int, 10> rm = fr;
    for (int d = 0; d < 10; d++) rm[d] -= cntS[y0][d];
    string pr = "";
    int pl = n - tl.size();
    if (pl == 1)
    {
        for (int d = 0; d < 10; d++)
        {
            if (rm[d])
            {
                pr += '0' + d;
                break;
            }
        }
    }
    else
    {
        for (int d = 1; d < 10; d++)
        {
            if (rm[d])
            {
                pr += '0' + d;
                rm[d]--;
                break;
            }
        }
        for (int d = 0; d < 10; d++)
            for (int j = 0; j < rm[d]; j++)
                pr += '0' + d;
    }
    
    cout << pr + tl << '\n';
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    pre();
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
}
```
