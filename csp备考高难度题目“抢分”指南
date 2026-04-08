# 🏆 CCF-CSP T4/T5 高难度题目“抢分”指南

在 CSP 考试中，T4 和 T5 的满分往往需要极高的算法造诣（如线段树、平衡树、动态 DP 等）。但为了达到 **300分** 的目标，我们的核心战略是：**放弃正解，精准识别数据范围，用最简单的暴力代码拿走 20~60 分！**

以下是根据题目给定的**数据范围（子任务）**划分的抢分套路与 C++ 模板：

---

## 1. 看到 $$N \le 10$$ 或 $$N \le 11$$：全排列暴力
**适用场景**：题目要求寻找一种“最优的操作顺序”、“最短的遍历路径”等。
**核心武器**：`std::next_permutation` ($$O(N!)$$ 复杂度)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    int n = 10;
    vector<int> a(n);
    for(int i = 0; i < n; i++) a[i] = i + 1; // 初始化为 1~N

    // 必须先排序，确保从最小字典序开始！
    sort(a.begin(), a.end()); 

    int ans = 2e9; // 找最小值初始化为极大值
    do {
        int current_cost = 0;
        // --- 在这里写你的 $O(N)$ 模拟逻辑 ---
        // 按照当前 a 数组的顺序去执行操作，算出花费
        
        ans = min(ans, current_cost);
    } while (next_permutation(a.begin(), a.end()));

    cout << ans << endl;
    return 0;
}
```

---

## 2. 看到 $$N \le 20$$：二进制子集枚举
**适用场景**：题目要求“从 $$N$$ 个物品中选出若干个”、“每个节点有选或不选两种状态”，求最优解（如背包问题变种、最大独立集）。
**核心武器**：位运算（Bitwise Operations） ($$O(2^N)$$ 复杂度)
**原理解析**：用一个整数的二进制位表示状态。例如 $$N=3$$，数字 `5` 的二进制是 `101`，表示选中第 0 个和第 2 个物品。

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    int n = 20;
    int max_state = 1 << n; // 2^n 种状态，从 0 到 2^n - 1
    
    int best_ans = 0;
    // 遍历所有可能的选择方案
    for (int state = 0; state < max_state; state++) {
        int current_val = 0;
        bool is_valid = true;
        
        // 检查当前方案 state 中选了哪些物品
        for (int i = 0; i < n; i++) {
            if (state & (1 << i)) { // 如果 state 的第 i 位是 1
                // 物品 i 被选中了，加上它的价值或判断冲突
                // current_val += value[i];
            }
        }
        
        if (is_valid) {
            best_ans = max(best_ans, current_val);
        }
    }
    cout << best_ans << endl;
    return 0;
}
```

---

## 3. 看到 $$N, Q \le 2000$$：双层循环暴力模拟
**适用场景**：题目给了一个数组/树/图，有 $$Q$$ 次操作，每次操作修改一个区间，或者查询一个区间的信息。
**正解往往是**：线段树、树状数组。
**抢分战术**：直接开普通数组，用 `for` 循环硬改、硬查！($$O(N \times Q)$$ 复杂度)

```cpp
#include <iostream>
#include <vector>
using namespace std;

const int MAXN = 2005;
int a[MAXN]; // 普通数组

int main() {
    int n, q;
    cin >> n >> q;
    
    while (q--) {
        int type, L, R, val;
        cin >> type;
        if (type == 1) {
            // 操作 1：区间修改 (例如区间加 val)
            cin >> L >> R >> val;
            for (int i = L; i <= R; i++) {
                a[i] += val; // 暴力修改
            }
        } else if (type == 2) {
            // 操作 2：区间查询 (例如求区间和)
            cin >> L >> R;
            long long sum = 0;
            for (int i = L; i <= R; i++) {
                sum += a[i]; // 暴力统计
            }
            cout << sum << endl;
        }
    }
    return 0;
}
```

---

## 4. 看到图论题 $$N \le 300$$：Floyd 算法
**适用场景**：任意两点间的最短路径问题。
**正解往往是**：多次 Dijkstra 或者复杂的图论建模。
**抢分战术**：只需 4 行代码的 Floyd 算法！($$O(N^3)$$ 复杂度)

```cpp
#include <iostream>
#include <vector>
using namespace std;

const int INF = 1e9;
int dist[305][305];

int main() {
    int n, m;
    cin >> n >> m;
    
    // 初始化
    for(int i=1; i<=n; i++)
        for(int j=1; j<=n; j++)
            dist[i][j] = (i == j) ? 0 : INF;
            
    // 读入边
    for(int i=0; i<m; i++){
        int u, v, w;
        cin >> u >> v >> w;
        dist[u][v] = min(dist[u][v], w);
        dist[v][u] = min(dist[v][u], w); // 无向图
    }
    
    // Floyd 核心 4 行代码 (注意 k 必须在最外层！)
    for (int k = 1; k <= n; k++) {
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    
    // 此时 dist[i][j] 就是任意两点间的最短路
    return 0;
}
```

---

## 5. 树上问题的“特殊性质”抢分
很多 T4/T5 是树上问题，题目最后会给出“特殊性质”的子任务，通常占 10~20 分。
*   **特殊性质 A：树退化成一条链 (每个点最多2个度)**
    *   **战术**：树的问题瞬间变成了**一维数组**问题！直接用前缀和或者一维数组的暴力循环解决。
*   **特殊性质 B：树是一个“菊花图” (存在一个中心点连向所有其他点)**
    *   **战术**：任意两点的距离最多只有 2！直接围绕中心点写特判逻辑（`if-else`），完全不需要写 DFS/LCA。

---

## 💡 终极考场策略：
1. **绝不交白卷**：如果连暴力都不会写，看看题目是不是要求输出 `YES/NO`，或者输出一个数字。直接 `cout << "YES" << endl;` 或者 `cout << -1 << endl;`，有时候能白嫖 5 分！
2. **时间分配**：在 T4/T5 上写暴力代码的时间**绝对不能超过 40 分钟**。写完、测过样例、交上去，立刻回头去检查 T1、T2 的边界条件，或者死磕 T3 的大模拟！
