# 🏆 最短路算法经典模板与核心原理解析

在算法竞赛中，直接套用以下结构即可。代码下方附带了面试和比赛中最常考的核心细节解析。

---

## 1. SPFA (Shortest Path Faster Algorithm) 

**一句话总结**：每一次会检测所有的邻居，如果有需要更新的就先更新，然后再把邻居入队列。
**适用场景**：单源最短路，图中**包含负权边**（但不能有负权环）。
**时间复杂度**：平均 $$O(km)$$，最坏 $$O(nm)$$。

### 💻 标准模板
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;        // 最大节点数
const int INF = 0x3f3f3f3f;     // 定义无穷大

struct Edge {
    int to;
    int weight;
};

vector<Edge> adj[MAXN]; // 邻接表存图
int dist[MAXN];         // 存储从起点到各个点的最短距离
bool in_q[MAXN];        // 标记节点是否在队列中

// start: 起点, n: 总节点数
void spfa(int start, int n) {
    // 1. 初始化
    for (int i = 1; i <= n; i++) {
        dist[i] = INF;      
        in_q[i] = false;    
    }
    
    queue<int> q;
    
    // 2. 起点入队
    dist[start] = 0;
    q.push(start);
    in_q[start] = true;
    
    // 3. 核心队列更新
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        in_q[u] = false; // ⚠️ 出队必须立刻标记为不在队列中！

        // 遍历 u 的所有邻居
        for (auto edge : adj[u]) {
            int v = edge.to;
            int w = edge.weight;
            
            // ⚠️ 松弛操作：如果借由 u 走到 v 更近
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w; 
                
                // 如果 v 不在队列中，就让 v 入队去通知它的邻居
                if (!in_q[v]) {
                    q.push(v);
                    in_q[v] = true;
                }
            }
        }
    }
}
```

### 🧠 核心细节解析
1. **为什么无穷大要用 `0x3f3f3f3f` 而不是 `INT_MAX`？**
   - 在执行 `dist[u] + w < dist[v]` 时，如果 `dist[u]` 是 `INT_MAX`，加上正数 `w` 会导致**整型溢出**变成负数，算法会误以为找到了极短的路径而崩溃。
   - `0x3f3f3f3f` 足够大（十亿级别），且两个 `0x3f3f3f3f` 相加也不会超过 `int` 上限，完美避开溢出。
2. **`in_q` 数组的作用是什么？**
   - 类似“村长广播”：如果一个人的距离被更新了，他就需要拿大喇叭（进队列）去通知邻居。
   - 如果他**已经在排队**准备广播了（`in_q[v] == true`），我们就只更新他的距离，不让他重复排队，避免浪费时间。出队广播完后，必须立刻标为 `false`，因为以后他可能还会被更新并重新排队。

---

## 2. Floyd-Warshall 

**适用场景**：多源最短路（求任意两点间的最短路），图极小（$$n \le 500$$），允许有负权边。
**时间复杂度**：$$O(n^3)$$。

### 💻 标准模板
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 505;           
const int INF = 0x3f3f3f3f;

int dist[MAXN][MAXN]; // dist[i][j] 表示 i 到 j 的最短距离

// n: 总节点数
void floyd(int n) {
    // ⚠️ 核心代码只有短短 5 行
    // k 必须在最外层循环！k 代表“允许经过的中转点集合”
    for (int k = 1; k <= n; k++) {
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                // 防止两个 INF 相加导致整型溢出，或负权边导致误判
                if (dist[i][k] != INF && dist[k][j] != INF) {
                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
                }
            }
        }
    }
}

// 初始化方式：
void init(int n) {
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            if (i == j) dist[i][j] = 0;   
            else dist[i][j] = INF;        
        }
    }
    // 之后再读取图的边，更新 dist[u][v] = weight
}
```

### 🧠 核心细节解析
1. **为什么 $$k$$ 必须在最外层循环？（极其重要）**
   - Floyd 的本质是**动态规划**。状态 $$dp[k][i][j]$$ 表示：只允许经过前 $$k$$ 个节点作为中转站的情况下，$$i$$ 到 $$j$$ 的最短距离。
   - 正确逻辑是**逐步解锁中转站**：当 $$k=1$$ 时，检查所有人能否通过 1 号点缩短距离；当 $$k=2$$ 时，检查所有人能否通过 2 号点缩短距离。
   - 如果把 $$i, j$$ 放外层，意味着固定了起点和终点去尝试所有中转站。但你在尝试用 $$k$$ 中转时，$$i$$ 到 $$k$$ 的最短路可能还没被完全算好（因为有些中转站还没被遍历到），会导致用半成品更新状态，结果全错。
2. **为什么需要 `if (dist[i][k] != INF && dist[k][j] != INF)`？**
   - 在有**负权边**的图中，如果 $$i$$ 到 $$k$$ 不通（`INF`），而 $$k$$ 到 $$j$$ 的边权是 $$-5$$。如果不加判断，`INF + (-5)` 会变成一个比 `INF` 略小的数，系统会误以为 $$i$$ 和 $$j$$ 连通了。加上判断能保证只有真正连通的路才参与中转。
