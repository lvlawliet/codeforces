# Tree Orientation (Easy Version)

## 1. 题目翻译 (Translation)

### 题目背景
你原本有一棵包含 $n$ 个节点的无向树。为了让它更有趣，你给这 $n-1$ 条边分别指定了一个方向。
现在你忘记了树的结构和边的方向，但你有一份记录，上面写着在方向确定后，对于所有有序对 $(u, v)$，节点 $u$ 是否可以到达节点 $v$。

### 任务
根据给定的可达性信息（邻接矩阵形式），判断是否存在符合条件的树结构和边定向。如果存在，构造并输出其中一种方案。

### 输入格式
* $t$：测试用例数量 ($1 \le t \le 10^4$)。
* $n$：节点数量 ($2 \le n \le 500$)。
* $n$ 行长度为 $n$ 的字符串：第 $i$ 行第 $j$ 个字符为 '1' 表示 $i$ 可达 $j$，否则不可达。
* 保证所有测试用例的 $n^3$ 之和不超过 $500^3$。

---

## 2. 解题思路 (Approach)

在树中，任意两个节点之间的路径是唯一的。如果 $u$ 能够到达 $v$，那么连接它们的唯一路径必须全部朝向 $v$。

### 关键概念：传递归约 (Transitive Reduction)
对于一个有向无环图（DAG），如果存在一条边 $u \to v$，并且不存在另一个节点 $w$ 满足 $u \to \dots \to w \to \dots \to v$，那么这条边就是**不可或缺**的。
在树的结构中，传递闭包的**传递归约**结果就是原本的那棵树。

### 解题步骤
1.  **验证合法性（必要条件）**：
      * **无环性**：如果 $i$ 能到 $j$ 且 $j$ 能到 $i$ （其中 $i \neq j$ ），则存在环，不可能是树。
    * **传递性**：如果 $i \to j$ 且 $j \to k$，那么必须有 $i \to k$。
2.  **提取候选边**：
    遍历所有满足 $R[i][j] = 1$ ($i \neq j$) 的点对。对于每一对 $(i, j)$，检查是否存在中间点 $k$ ($k \neq i, j$) 满足 $R[i][k] = 1$ 且 $R[k][j] = 1$。
    * 如果没有这样的中间点 $k$，说明 $i \to j$ 是一条直接相邻的边。
3.  **验证构造结果**：
    * 得到的边数必须正好等于 $n - 1$。
    * 这 $n - 1$ 条边在无向意义下必须构成一棵连通树（可以使用 并查集 DSU 检查）。
    * 由这些边生成的传递闭包必须与输入的矩阵完全一致。

### 复杂度
* 对于每个测试用例，寻找候选边的时间复杂度为 $O(n^3)$。
* 总复杂度 $\sum n^3 \le 500^3$，在 3 秒的时间限制内绰绰有余。

---

## 3. C++ 代码实现 (Implementation)

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <numeric>

using namespace std;

struct DSU
{
    vector<int> parent;
    DSU(int n)
    {
        parent.resize(n + 1);
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int i)
    {
        if (parent[i] == i)
            return i;
        return parent[i] = find(parent[i]);
    }
    void unite(int i, int j)
    {
        int root_i = find(i), root_j = find(j);
        if (root_i != root_j)
            parent[root_i] = root_j;
    }
};

void solve()
{
    int n;
    cin >> n;
    vector<string> r(n);
    for (int i = 0; i < n; ++i)
        cin >> r[i];

    for (int i = 0; i < n; ++i)
    {
        if (r[i][i] == '0')
        {
            cout << "No" << endl;
            return;
        }
        for (int j = 0; j < n; ++j)
        {
            if (i == j)
                continue;
            if (r[i][j] == '1' && r[j][i] == '1')
            {
                cout << "No" << endl;
                return;
            }
        }
    }

    vector<pair<int, int>> edges;
    for (int i = 0; i < n; ++i)
    {
        for (int j = 0; j < n; ++j)
        {
            if (i == j || r[i][j] == '0')
                continue;

            bool has_intermediate = false;
            for (int k = 0; k < n; ++k)
            {
                if (k == i || k == j)
                    continue;
                if (r[i][k] == '1' && r[k][j] == '1')
                {
                    has_intermediate = true;
                    break;
                }
            }
            if (!has_intermediate)
            {
                edges.push_back({i + 1, j + 1});
            }
        }
    }

    if (edges.size() != n - 1)
    {
        cout << "No" << endl;
        return;
    }

    DSU dsu(n);
    for (auto &e : edges)
        dsu.unite(e.first, e.second);

    int components = 0;
    for (int i = 1; i <= n; ++i)
        if (dsu.parent[i] == i)
            components++;
    if (components > 1)
    {
        cout << "No" << endl;
        return;
    }

    vector<vector<int>> dist(n, vector<int>(n, 0));
    for (int i = 0; i < n; i++)
        dist[i][i] = 1;
    for (auto &e : edges)
        dist[e.first - 1][e.second - 1] = 1;

    for (int k = 0; k < n; k++)
    {
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < n; j++)
            {
                if (dist[i][k] && dist[k][j])
                    dist[i][j] = 1;
            }
        }
    }

    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < n; j++)
        {
            if (dist[i][j] != (r[i][j] - '0'))
            {
                cout << "No" << endl;
                return;
            }
        }
    }

    cout << "Yes" << endl;
    for (auto &e : edges)
    {
        cout << e.first << " " << e.second << endl;
    }
}

int main()
{
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;
    while (t--)
        solve();
    return 0;
}
```

# Tree Orientation (Hard Version)

## 1. 题目翻译 (Translation)

### 题目背景
和Easy没区别，唯一改变的是 $n$：节点数量 ($2 \le n \le 8000$)

---

## 2. 核心优化思路

在有向树中，对于任意节点 $u$ ，它指向的直接子节点 $v_1, v_2, \dots$ 所能到达的节点集合（子树）是 **互不相交** 的。

### 算法逻辑：
1.  **预处理位图**：使用 `std::bitset<8000>` 存储每个节点的出达集合 $B_u$（即矩阵的每一行）。
2.  **出度排序**：计算每个节点的可达节点总数（即位图中 $1$ 的个数）。
3.  **贪心提取边**：
    对于每一个节点 $u$ ：
    * 找到所有 $u$ 可达的节点集合 $S = \{v \mid R[u][v] = 1, v \neq u\}$ 。
    * 将 $S$ 中的节点按其 **出度从大到小** 排序。
    * 维护一个位图 `covered` ，初始为空。
    * 按排序顺序遍历 $v \in S$ ：
        * 如果 $v$ 不在 `covered` 中，说明从 $u$ 到 $v$ 没有经过任何已经发现的中间点。
        * 判定 $(u, v)$ 为一条 **直接边** 。
        * 将 $v$ 的位图 $B_v$ 合并（OR 运算）到 `covered` 中。

### 为什么这样做是对的？
由于是树结构，如果 $u \to v$ 是间接可达（即 $u \to w \to \dots \to v$ ），那么 $w$ 的出度一定比 $v$ 大（因为 $w$ 的可达集合包含 $v$ 及其子树）。通过按出度降序排列，我们会先处理“更近”的中间点 $w$ ，从而将“更远”的 $v$ 标记为已覆盖。

---

## 3. C++ 代码实现 (Implementation)

``` cpp
#include <iostream>
#include <vector>
#include <string>
#include <bitset>
#include <algorithm>

using namespace std;

const int MAXN = 8005;
bitset<MAXN> M[MAXN];
bitset<MAXN> edges_bs[MAXN];
int sz[MAXN];
int P[MAXN];

void solve() 
{
    int n;
    cin >> n;
    
    for (int i = 0; i < n; ++i) 
    {
        string s;
        cin >> s;
        M[i].reset();
        for (int j = 0; j < n; ++j) 
        {
            if (s[j] == '1') 
            {
                M[i].set(j);
            }
        }
        sz[i] = M[i].count(); 
        P[i] = i;
    }

    sort(P, P + n, [](int a, int b){return sz[a] > sz[b];});

    int edge_count = 0;
    for (int i = 0; i < n; ++i) 
    {
        edges_bs[i] = M[i];
        edges_bs[i].reset(i); 
    }

    for (int i = 0; i < n; ++i) 
    {
        for (int k = 0; k < n; ++k)
        {
            int j = P[k];
            if (edges_bs[i].test(j)) 
            {
                edges_bs[i] &= ~M[j]; 
                edges_bs[i].set(j);   
                edge_count++;
            }
        }
        if (edge_count > n) break; 
    }

    
    if (edge_count != n - 1) 
    {
        cout << "No\n";
        return;
    }

    vector<int> parent(n);
    for(int i = 0; i < n; ++i) parent[i] = i;
    auto find_set = [&](auto& self, int i) -> int 
    {
        return parent[i] == i ? i : (parent[i] = self(self, parent[i]));
    };

    vector<pair<int, int>> tree_edges;
    vector<vector<int>> adj(n);
    bool possible = true;

    for (int i = 0; i < n; ++i) 
    {
        for (int j = 0; j < n; ++j) 
        {
            if (edges_bs[i].test(j)) 
            {
                tree_edges.push_back({i, j});
                int root_i = find_set(find_set, i);
                int root_j = find_set(find_set, j);
                if (root_i != root_j) 
                {
                    parent[root_i] = root_j;
                } else 
                {
                    possible = false; 
                }
                adj[i].push_back(j);
            }
        }
    }

    if (!possible || tree_edges.size() != n - 1) 
    {
        cout << "No\n";
        return;
    }

    for (int i = 0; i < n; ++i) 
    {
        bitset<MAXN> reach;
        vector<int> q;
        q.push_back(i);
        reach.set(i);
        int head = 0;
        
        while (head < (int)q.size()) 
        {
            int u = q[head++];
            for (int v : adj[u]) 
            {
                if (!reach.test(v)) 
                {
                    reach.set(v);
                    q.push_back(v);
                }
            }
        }
        
        if (reach != M[i]) 
        {
            cout << "No\n";
            return;
        }
    }

    cout << "Yes\n";
    for (auto p : tree_edges) 
    {
        cout << p.first + 1 << " " << p.second + 1 << "\n";
    }
}

int main() 
{
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    
    int t;
    if (cin >> t) 
    {
        while (t--) 
        {
            solve();
        }
    }
    return 0;
}
```
