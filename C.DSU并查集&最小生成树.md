# 🌳 并查集 (DSU) 与最小生成树 (MST) 核心笔记

---

## 一、 基础并查集 (Standard DSU)

并查集主要用于处理不相交集合的合并与查询问题。

```cpp
const int N = 10005;
int fa[N]; // fa[i] 存储 i 的直接上级

// 【初始化】
// 每个人最开始都是自己的老大（孤立点）
void init(int n) {
    for (int i = 1; i <= n; i++) {
        fa[i] = i; 
    }
}

// 【查询 + 路径压缩】(重点！)
// 疑惑解答：为什么写成 return fa[x] = find(fa[x])？
// 逻辑拆解：
// 1. find(fa[x]): 先递归向上找，直到找到最终的“祖宗”。
// 2. fa[x] = ...: 把 x 的直接上级改为这个“祖宗”。(这就是路径压缩，下次再查 x 就是 O(1) 了)
// 3. return ... : 返回祖宗的编号。
// 作用：把长长的“链表”结构瞬间拍扁，让所有人直接挂在祖宗下面。
int find(int x) {
    if (x == fa[x]) return x; // 递归出口：如果上级是自己，那自己就是祖宗
    return fa[x] = find(fa[x]); // 注意不是 find(x)，否则会死循环
    // 等价于赋值之后再返回新的 fa[x]
}

// 【合并】
void join(int x, int y) {
    int fx = find(x);
    int fy = find(y);
    if (fx != fy) {
        fa[fx] = fy; // 让 x 的老大认 y 的老大做上级
    }
}
```

---

## 二、 扩展并查集 (种类并查集)

面对处理“敌人的敌人是朋友”这种问题时，使用扩展并查集（开多倍空间）。

```cpp
const int N = 20005; // 【注意】空间要开 2 倍！(N + N)
int fa[N]; 

// 【初始化】
// 无论是本体还是影子，最开始都是独立的
void init(int n) {
    // 注意循环范围是 2*n
    for (int i = 1; i <= 2 * n; i++) {
        fa[i] = i; 
    }
}

// 【主体部分】
void solve(int n, int m) {
    init(n);
    
    for(int i = 0; i < m; i++) {
        char type; 
        int u, v;
        cin >> type >> u >> v;
        
        if (type == 'F') { 
            // 情况1：u 和 v 是朋友
            // 逻辑：朋友的朋友是朋友，朋友的敌人也是敌人
            join(u, v);           // 1. 本体连本体
            join(u + n, v + n);   // 2. 影子连影子 (关键！别漏了)
        } 
        else { 
            // 情况2：u 和 v 是敌人
            // 逻辑：敌人的敌人是朋友
            // u 的敌人域(u+n) 就是 v 的朋友域(v)
            join(u + n, v);       // 1. 我的影子 连 你的本体
            join(u, v + n);       // 2. 我的本体 连 你的影子
        }
    }
}
```

---

## 三、 边结构体与排序重载

含有加权的问题时，可以使用结构体封装边，并重载 `<` 运算符方便排序。这在二分图和最小生成树中极为常用。

```cpp
struct Edge {
    int u, v; // 连接的两个点
    int w;    // 权值 (weight)

    // 【重载小于号 <】(重点！)
    // 作用：告诉 std::sort 怎么比较两个 Edge
    // 语法解析：
    // 1. const Edge &other : 引用传递，避免拷贝，加 const 防止误改
    // 2. const (函数尾) : 保证这个函数不会修改自身的数据
    bool operator<(const Edge &other) const {
        return w < other.w; // 按权值【从小到大】排序
        // 如果要从大到小，改成 return w > other.w;
    }
};
```

---

## 四、 最小生成树 (Kruskal 算法)

**核心思想**：贪心 + 并查集。
将所有边按权值从小到大排序，依次尝试加入图中。如果这条边的两个端点**不在同一个集合**（用并查集判断，防止形成环），就加入这条边，直到加入了 $n-1$ 条边为止。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 10005;
int fa[MAXN];

// 1. 结构体定义 (直接复用上面的 Edge)
struct Edge {
    int u, v, w;
    bool operator<(const Edge &other) const {
        return w < other.w;
    }
};

// 2. 并查集基础函数
void init(int n) {
    for (int i = 1; i <= n; i++) fa[i] = i;
}

int find(int x) {
    if (x == fa[x]) return x;
    return fa[x] = find(fa[x]);
}

// 3. Kruskal 主逻辑
int kruskal(int n, vector<Edge>& edges) {
    init(n);
    
    // 步骤 1：将所有边按权值从小到大排序
    sort(edges.begin(), edges.end());
    
    int total_weight = 0; // 记录最小生成树的总权值
    int edge_count = 0;   // 记录已加入的边数
    
    // 步骤 2：从小到大遍历所有边
    for (auto edge : edges) {
        int u = edge.u, v = edge.v, w = edge.w;
        
        int fu = find(u), fv = find(v);
        
        // 如果两个点不在同一个集合中（即加入这条边不会形成环）
        if (fu != fv) {
            fa[fu] = fv;           // 合并集合
            total_weight += w;     // 累加权值
            edge_count++;          // 边数 +1
            
            // 优化：如果已经加入了 n-1 条边，树已经生成完毕，提前退出
            if (edge_count == n - 1) break;
        }
    }
    
    // 步骤 3：检查图是否连通
    if (edge_count < n - 1) {
        return -1; // 无法构成生成树（图不连通）
    }
    
    return total_weight;
}
```
