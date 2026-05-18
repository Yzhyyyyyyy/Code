# 背包问题动态规划 (Knapsack DP) 总结

背包问题是动态规划中的经典模型。核心状态定义通常为：**`dp[i][j]` 表示在前 `i` 个物品中选，背包容量为 `j` 时的最大价值（或方案数、可行性等）**。

通过空间优化，通常可以压缩为一维数组 `dp[j]`，其含义为：**只考虑拥有 `j` 的容量/预算时，能得到的最大价值**。

---

## 1. 01 背包 (01 Knapsack)
**特点**：每个物品最多只能取一次。
**核心**：枚举容量时**必须倒序**，防止同一物品在同一轮被多次选取。

```cpp
// w[i]: 重量, v[i]: 价值, V: 背包总容量
vector<int> dp(V + 1, 0);

for (int i = 0; i < N; i++) {           // 1. 枚举物品
    for (int j = V; j >= w[i]; j--) {   // 2. 【重点】枚举容量：必须倒序！
        dp[j] = max(dp[j], dp[j - w[i]] + v[i]);
    }
}
```

---

## 2. 完全背包 (Complete Knapsack)
**特点**：每个物品可以无限次选取。
**核心**：枚举容量时**改为正序**，允许当前物品的状态基于本轮已更新的状态转移。

```cpp
for (int i = 0; i < N; i++) {           // 1. 枚举物品
    for (int j = w[i]; j <= V; j++) {   // 2. 【重点】枚举容量：正序！
        dp[j] = max(dp[j], dp[j - w[i]] + v[i]);
    }
}
```

---

## 3. 多重背包 (Multiple Knapsack)
**特点**：每个物品有数量限制（有限个）。
**核心**：**二进制拆分**。将数量为 $C$ 的物品按 $1, 2, 4, \dots, 2^k, C - 2^k + 1$ 拆分成多个只能取一次的新物品，然后跑 01 背包。

```cpp
struct Item { int w, v; };
vector<Item> new_items;

// 假设当前物品重量为 w, 价值为 v, 数量为 count
for (int k = 1; k <= count; k *= 2) {
    new_items.push_back({k * w, k * v});
    count -= k;
}
if (count > 0) { // 把剩下的打包
    new_items.push_back({count * w, count * v});
}
// 最后对 new_items 跑一遍标准的 01 背包
```

---

## 4. 分组背包 (Group Knapsack)
**特点**：物品被划分为多个组，**每组内最多只能选一个物品**。
**核心循环顺序**：遍历组 $\to$ 倒序遍历容量 $\to$ 遍历组内物品。

```cpp
int start = 0;
while (start < n) {
    // 1. 划分当前组 [start, end)
    int end = start;
    while (end < n && num[end].group == num[start].group) end++;
    
    // 2. 分组背包核心转移
    for (int j = max_weight; j >= 0; j--) { // 【重点】倒序遍历背包容量
        for (int k = start; k < end; k++) { // 遍历当前组的所有物品
            if (j >= num[k].weight) {
                dp[j] = max(dp[j], dp[j - num[k].weight] + num[k].value);
            }
        }
    }
    start = end; // 移到下一组
}
```

---

## 5. 主附件背包 / 依赖背包 (Dependency Knapsack)
**特点**：选择附件的前提是必须选择对应的主件。
**核心**：将主件和其附件的所有组合情况枚举出来，视作一个“组”，转化为**分组背包**。

```cpp
// 1. 排序：同组放一起，主件排在最前面 (q=0)
sort(num.begin() + 1, num.end(), cmp);

int start = 1;
while (start <= n) {
    int end = start + 1;
    while (end <= n && num[end].family == num[start].family) end++;
    int accessories_cnt = end - start - 1; 
    
    // 2. 二进制枚举当前主件+附件的所有合法组合
    vector<pair<int, int>> combos;
    for (int i = 0; i < (1 << accessories_cnt); i++) {
        int price_now = num[start].price;
        int value_now = num[start].value;
        for (int j = 0; j < accessories_cnt; j++) {
            if ((i >> j) & 1) { // 选中第 j 个附件
                price_now += num[start + 1 + j].price;
                value_now += num[start + 1 + j].value;
            }
        }
        if (price_now <= max_price) combos.push_back({price_now, value_now});
    }

    // 3. 跑分组背包 (倒序容量 -> 遍历组合)
    for (int j = max_price; j >= 0; j--) {
        for (auto &combo : combos) {
            if (j >= combo.first) {
                dp[j] = max(dp[j], dp[j - combo.first] + combo.second);
            }
        }
    }
    start = end;
}
```

---

## 6. 余数背包 (Remainder Knapsack)
**特点**：求选物品总和模 $F$ 余指定值的方案数（或最值）。
**核心**：一维数组直接更新会导致状态相互覆盖（0余数自更新等），必须使用**临时数组进行物理隔离**。

```cpp
vector<int> dp(f, 0); // dp[j]：模 f 余 j 的方案数
dp[0] = 1; // 初始化空集

for (int i = 0; i < n; i++) {
    int r = num[i] % f; 
    vector<int> temp = dp; // 【重点】复制旧状态，隔离本轮更新
    
    for (int j = 0; j < f; j++) { 
        int pre_j = (j - r + f) % f; // 修正负数下标
        temp[j] = (temp[j] + dp[pre_j]) % mod; // 基于旧状态 dp 更新 temp
    }
    dp = temp; // 覆盖回原数组
}
// 最终结果排除空集：(dp[0] - 1 + mod) % mod
```

---

## 7. 可行性背包 (Feasibility Knapsack)
**特点**：不求最大价值，只求某个容量/状态**是否能被凑出**。
**核心**：将 `int` 数组换成 `bool` 数组，状态转移方程中的 `max` 替换为逻辑或 `||`。

```cpp
// dp[j] 表示容量 j 是否可达
vector<bool> dp(V + 1, false);
dp[0] = true;

for (int i = 0; i < N; i++) {
    for (int j = V; j >= w[i]; j--) {
        dp[j] = dp[j] || dp[j - w[i]];
    }
}
```
