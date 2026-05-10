# 🧭 DFS 核心思想与实战模板笔记

## 一、 DFS 的灵魂拷问：到底要不要回溯？

在写 DFS 时，最容易犯的致命错误就是“乱加回溯”或“忘加回溯”。记住以下核心准则：

### 1. 【必须回溯】的场景：找“所有方案” / 走迷宫找“所有路径”
- **核心思想**：`visited` 数组代表**“当前这条路径有没有踩过这个点”**。
- **原因**：为了尝试所有可能的组合，当你碰壁或找到一条有效路径后，退回来时**必须擦除脚印（回溯）**，否则会挡住其他路径。
- **典型题型**：迷宫寻路、N 皇后、排列组合、全排列。

### 2. 【绝对不能回溯】的场景：找“最优解” / “最短路” / “最小代价”
- **核心思想**：`visited`（通常叫 `min_cost` 或 `dist`）数组代表**“到达当前点的历史最优成绩”**。
- **原因**：如果你把历史最优成绩撤销了，后续更差的路径来到这里时，就会因为没有“排行榜”的阻挡而继续往下傻搜，导致程序退化成指数级复杂度，**必定 TLE**！
- **典型题型**：DFS 版 SPFA、带记忆化的 DFS、求路径最大边权的最小值。

---

## 二、 模板库 1：必须回溯（找所有方案）

### 1. 迷宫寻路模板
**💡 考场避坑**：在 C++ 中，尽量不要在函数参数里写 `int num[][m]` 这种变长数组(VLA)，极易引发编译错误或段错误。**强烈建议将迷宫数组开成全局变量！**

```cpp
#include <iostream>
using namespace std;

// 定义方向数组方便移动 (右, 下, 左, 上)
int dx[4] = {0, 1, 0, -1};
int dy[4] = {1, 0, -1, 0};

// 假设迷宫和路径数组开在全局
int num[1005][1005]; 
int path[1005][1005];

// 检查是否越界且可走
bool islaw(int tx, int ty, int n, int m) {
    return (tx >= 0 && tx < n && ty >= 0 && ty < m && num[tx][ty] == 1 && path[tx][ty] == 0);
}

void DFS(int x, int y, int n, int m) {
    // 1. 终点判定：如果完成一条路径，函数返回
    if (x == n - 1 && y == m - 1) {
        // 进行操作（打印路径、方案数 +1 等）
        return;
    }
    
    // 2. 标记该位置并移动，进行递推
    for (int d = 0; d < 4; d++) { // 重点：确保每个位置检查了四个方向
        int tx = x + dx[d];
        int ty = y + dy[d];
        
        if (islaw(tx, ty, n, m)) {
            path[tx][ty] = 1;  // 标记点位已经走到
            DFS(tx, ty, n, m); // 递归深入
            
            // 【核心】：回溯！擦除脚印，尝试其他方向
            path[tx][ty] = 0;  
            // TIP: 提前结束也要回溯，使这一段没有产生改变
        }
    }
}
```

### 2. N 皇后问题模板
**💡 核心技巧**：使用 `x - y + n` 将副对角线的坐标映射为正数下标，完美解决负数越界问题！

```cpp
// 检查列、主对角线、副对角线是否被占用
bool law(int x, int y, int n, int num[30][4]) {
    return (num[y][1] == 0 && num[x - y + n][2] == 0 && num[x + y][3] == 0);
}

void dfs(int x, int *di, int n, int result[][13], int num[30][4], int current[13]) {
    // 1. 终点判定：保存当前解
    if (x == n) {
        // 重点：必须用 current 数组暂存当前路径！
        // 如果没有 current 数组，每次尝试都会修改 result 中的值，导致输出错误
        for (int i = 0; i < n; i++) {
            result[*di][i] = current[i];
        }
        (*di)++;
        return;
    }
    
    // 2. 尝试在当前行 x 的每一列 y 放置皇后
    for (int y = 0; y < n; y++) {
        if (law(x, y, n, num)) {
            current[x] = y + 1;  // 记录当前选择
            
            // 标记占用
            num[y][1] = 1;
            num[x - y + n][2] = 1;
            num[x + y][3] = 1;
            
            dfs(x + 1, di, n, result, num, current);
            
            // 【核心】：回溯！拔掉皇后，尝试下一列
            num[y][1] = 0;
            num[x - y + n][2] = 0;
            num[x + y][3] = 0;
        }
    }
}
```

---

## 三、 模板库 2：绝对不能回溯（找最优解 / 最短路）

### 1. DFS 版最短路 / 最小代价（以“营救”求最大边权的最小值为例）
**💡 核心技巧**：利用 `min_cost` 数组记录历史最优解，进行强力剪枝。**一旦发现更优解才往下走，走完绝对不撤销！**

```cpp
#include <bits/stdc++.h>
using namespace std;

int n, m, s, t;
vector<pair<int,int>> road[10005];
int min_cost[10005]; // 记录到达每个点的历史最小拥挤度

void dfs(int from, int cost) {
    // 剪枝：如果当前代价已经比历史最优代价大，直接放弃这条路
    if (cost >= min_cost[from]) return; 
    
    // 终点判定
    if (from == t) return;

    for (auto i : road[from]) {
        int next_node = i.first;
        int edge_weight = i.second;
        
        // 计算走到下一个点的代价
        int next_cost = max(cost, edge_weight);
        
        // 【核心松弛操作】：只有发现更优解时，才更新并继续 DFS
        if (next_cost < min_cost[next_node]) {
            min_cost[next_node] = next_cost; // 记录最优解（打榜）
            dfs(next_node, next_cost);       // 继续往下搜
            
            // ❌ 绝对不要回溯！历史最优记录不能被撤销！
            // 错误写法：min_cost[next_node] = 恢复原值; 
        }
    }
}

int main() {
    // ... 读入数据建图 ...
    
    // 初始化所有点的历史最低代价为无穷大
    memset(min_cost, 0x3f, sizeof(min_cost));
    
    min_cost[s] = 0; // 起点代价为 0
    dfs(s, 0);
    
    cout << min_cost[t] << endl;
    return 0;
}
```
