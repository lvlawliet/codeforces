## 题目翻译

**传球**

体育课上，n 名学生排成一排，从左到右编号 1 到 n。

每个学生收到球后，会把球传给左边或右边的邻居，由字符串 s 决定：
- `s[i] = L`：第 i 个学生把球传给第 i-1 个学生
- `s[i] = R`：第 i 个学生把球传给第 i+1 个学生

第一个学生一定传右边，最后一个一定传左边（即 s 以 R 开头、L 结尾）。

**过程如下：**
- 首先，第 1 个学生拿到球
- 然后，恰好进行 n 次传球

求：**至少接到过一次球的学生数量**。

---

## 解题思路

由于 n ≤ 50，直接**模拟**整个传球过程即可：

1. 初始位置为第 0 个学生（下标从 0 开始）
2. 用一个 `visited` 集合或布尔数组记录哪些学生接过球
3. 将初始位置标记为已接球
4. 循环 n 次，根据当前学生的字符决定向左还是向右移动，并标记新位置
5. 最后统计标记过的学生数量

---

## C++ 代码

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <set>

using namespace std;

void solve() 
{
    int n;
    cin >> n;
    string s;
    cin >> s;

    set<int> visited;
    
    int current = 0;
    visited.insert(current + 1);

    for (int i = 0; i < n; ++i) 
    {
        if (s[current] == 'R') {

            current++;
        } else 
        {
            current--;
        }
        visited.insert(current + 1);
    }

    cout << visited.size() << endl;
}

int main() 
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int t;
    cin >> t;
    while (t--) 
    {
        solve();
    }
    return 0;
}
```
