## 题目翻译

**交替路径**

给定一个无向图，n 个顶点 m 条边，无自环无重边。

对每条边选择一个方向，将图变为有向图。定义**交替路径**为顶点序列 v₁,v₂,…,vₖ，满足：
- 边 (v₁,v₂) 方向：v₁ → v₂
- 边 (v₂,v₃) 方向：v₃ → v₂
- 边 (v₃,v₄) 方向：v₃ → v₄
- 边 (v₄,v₅) 方向：v₅ → v₄
- ……（出、入、出、入交替）

定义顶点 v 是**美丽的**，当且仅当：原图中所有从 v 出发的路径（可重复顶点）在有向图中都是交替路径。

求最多能使多少个顶点变得美丽？

---

## 解题思路

### 理解"美丽顶点"

交替路径的规律：奇数步**向外走**（out），偶数步**向内走**（in）。

若顶点 v 是美丽的，则：
- v 的所有邻居，从 v 出发的边必须是 **v → 邻居**（第1步出）
- 从邻居继续走，下一步必须是 **回到邻居**（第2步入）
- 以此类推……

这意味着：从 v 出发按 BFS/DFS 展开，**偶数层的点向外出发，奇数层的点向内收回**。

### 关键洞察：二部图

若一个连通分量是**二部图**（可二着色），则：

- 将图二着色为黑白两色
- **黑色层（偶数层）**：所有边方向朝外（→ 白色）
- **白色层（奇数层）**：所有边方向朝内（← 白色）

这样，**黑色顶点全部是美丽的**，白色顶点不是（因为从白色顶点出发第一步需要入边）。

若连通分量**不是二部图**（含奇环），则无法满足交替条件，**该连通分量中没有美丽顶点**。

> 孤立点（度为 0）无路径约束，天然是美丽的，单独计数。

### 算法步骤

1. 对每个连通分量做 BFS 二部图检测
2. 若是二部图：答案 += 较大色组的顶点数（贪心选多的一边）
3. 若不是二部图：贡献 0
4. 孤立点各贡献 1

---

## C++ 代码

```cpp
#include <bits/stdc++.h>
using namespace std;

void slove()
{
    int n, m;
    cin >> n >> m;

    vector<vector<int>> adj(n + 1);
    for (int i = 0; i < m; i++)
    {
        int u, v;
        cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    vector<int> color(n + 1, -1);
    int ans = 0;

    for (int start = 1; start <= n; start++)
    {
        if (color[start] != -1)
            continue;

        queue<int> q;
        q.push(start);
        color[start] = 0;

        int cnt[2] = {0, 0};
        bool isBipartite = true;

        while (!q.empty())
        {
            int u = q.front();
            q.pop();
            cnt[color[u]]++;

            for (int v : adj[u])
            {
                if (color[v] == -1)
                {
                    color[v] = 1 - color[u];
                    q.push(v);
                }
                else if (color[v] == color[u])
                {
                    isBipartite = false;
                }
            }
        }

        if (isBipartite)
        {
            ans += max(cnt[0], cnt[1]);
        }
    }

    cout << ans << "\n";
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int t;
    cin >> t;

    while (t--)
    {
        slove();
    }

    return 0;
}
```
