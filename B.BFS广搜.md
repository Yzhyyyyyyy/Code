# 🚀 BFS 与优先队列 (最短路) 核心笔记

## 一、 标准 BFS（适用于等权图 / 迷宫最短路）

### 1. 核心流程（五步走）
1. **初始化**：标记起点为已访问，将起点入队。
2. **循环遍历**：当队列非空时，出队队首节点，访问该节点。
3. **重点防坑**：一定要判断之前走过的地方不能再走（使用 `vis` 或 `path` 数组）。
4. **扩展节点**：遍历队首节点的所有合法邻接节点，**若未访问则标记为已访问并入队**。
5. **终止条件**：到达终点返回步数，或队列为空（所有可达节点遍历完毕）返回无解。

### 2. `std::queue` 常用 API
```cpp
queue<int> q; // 定义队列，存储节点编号或结构体

q.push(val);  // 队尾入队：将 val 加入队列尾部（扩展邻接节点时用）
q.pop();      // 队首出队：删除队列第一个元素（无返回值！处理完当前节点后弹出）
q.front();    // 获取队首：返回队首元素的引用（获取当前要处理的节点）
q.empty();    // 判空：返回 bool（BFS 循环条件 while(!q.empty()) 用）
q.size();     // 队列大小：返回队列中元素的个数
```
> ⚠️ **注意**：`q.pop()` 无返回值，仅用于删除队首元素。如果队列中需要存储多个维度的信息（如坐标+步数），请使用 `struct`。

### 3. 迷宫最短路标准模板
```cpp
#include <iostream>
#include <queue>
using namespace std;

struct locate {
    int x, y;
    int step; // 记录当前步数
    locate(int _x, int _y, int _s) : x(_x), y(_y), step(_s) {}
};

int dx[4] = {0, 1, 0, -1};
int dy[4] = {1, 0, -1, 0};

const int MAXN = 1005;
int num[MAXN][MAXN];  // 迷宫地图：1表示可走，0表示障碍
int path[MAXN][MAXN]; // 访问标记：1表示已走过，0表示未走过

// 判断坐标是否合法
bool is_valid(int tx, int ty, int n, int m) {
    return tx >= 0 && tx < n && ty >= 0 && ty < m && 
           num[tx][ty] == 1 && path[tx][ty] == 0;
}

int BFS(int start_x, int start_y, int n, int m) {
    queue<locate> q; // 队列 q 必须在函数内定义，避免多次调用的残留问题
    
    path[start_x][start_y] = 1; // 起点标记为已访问
    q.push(locate(start_x, start_y, 0));
    
    while (!q.empty()) {
        locate now = q.front(); // 获取队首元素
        q.pop();                // 弹出队首元素
        
        if (now.x == n - 1 && now.y == m - 1) { // 到达终点
            return now.step;
        }
        
        for (int i = 0; i < 4; i++) { // 四个方向扩展
            int tx = now.x + dx[i];
            int ty = now.y + dy[i];
            
            if (is_valid(tx, ty, n, m)) {
                path[tx][ty] = 1; // ⚠️ 关键：加入队列前立刻标记为已走过
                q.push(locate(tx, ty, now.step + 1));
            }
        }
    }
    return -1; // 无法到达终点
}
```

---

## 二、 优先队列（适用于带权图 / Dijkstra 雏形）

### 1. 为什么带权图不能用普通 BFS？
我们的目标是「找从起点到终点的最小花费路径」，关键看路径的「**总花费**」，而不是「**走了多少步**」：
- **无权图**：每一步花费相同（如走一步花 1 分钟）。普通 BFS 按“步数优先”，首次到达终点的必定是最短路径。
- **带权图**：每一步花费不同（如花费 0、1、2...）。此时步数少的路径总花费可能更大。必须使用优先队列，每次弹出「**当前总花费最小**」的节点进行扩展。

### 2. `std::priority_queue` 与结构体重载
`priority_queue` 本质是一个堆。默认情况下是**大顶堆**（每次弹出最大的元素）。为了实现找最小花费，我们需要将它改造成**小顶堆**。最优雅的做法是在 `struct` 内部重载 `<` 运算符。

### 3. 带权最短路结构体模板 (以 P3956 棋盘为例)
```cpp
#include <queue>
using namespace std;

struct node {
    int x, y;  // 核心1：当前所在的网格坐标
    int c;     // 核心2：当前坐标的颜色（存入结构体，避免频繁查数组）
    int w;     // 核心3：到达当前坐标的总花费（排序的核心依据）

    // ⚠️ 核心技巧：重载小于号，实现小顶堆
    // 记忆口诀：想要小顶堆，就用大于号 (>)
    bool operator<(const node& b) const { // 两个 const 都不能丢！
        return w > b.w; 
    }
};

// 定义优先队列，因为已经重载了 <，直接这样写就是小顶堆了！
priority_queue<node> q; 
```
