# 记忆化搜索 (DFS)：常见 TLE 踩坑与常数优化指南

在 CCF CSP 等算法竞赛中，状态转移方程写对了只是第一步。如果代码的**常数开销过大**，或者**记忆化机制存在漏洞**，依然会导致 TLE（Time Limit Exceeded）。

以下总结了记忆化搜索中最致命的几个性能杀手及优化方案：

---

## 坑点一：用 `double` 或 `int` 数组本身的默认值（0）来判断是否计算过（最致命！）

**❌ 你的写法：**
```cpp
// 依赖 dp 数组初始的 0 来判断
if (dp[now][num+1] != 0) 
    res += dp[now][num+1] * p[i];
```

**💥 为什么会 TLE？**
1. **逻辑漏洞**：在这道题中，当硬币攒够时，期望步数**本来就是 0**！如果某个状态真的是终点，DFS 计算完毕后返回了 `0`，存进 `dp` 数组里依然是 `0`。
2. **重复计算**：下次别的状态再需要这个终点时，一看 `dp` 还是 `0`，以为没算过，于是**又去调用了一次 DFS**！这导致终点状态被重复调用了无数次，记忆化彻底失效。
3. **浮点数精度**：用 `double` 类型直接判断 `!= 0` 在 C++ 中是非常危险的，容易因为精度问题导致判断失误。

**✅ 优化方案：引入专属的 `bool vis` 备忘录数组**
永远不要用 `dp` 数组的值本身去判断是否计算过，必须开一个同等大小的 `bool` 数组！
```cpp
bool vis[70000][85]; // 全局初始化默认为 false

// 在 DFS 中：
if (vis[now][num]) return dp[now][num]; // 只要算过，哪怕 dp 是 0 也直接返回！

// ... 计算过程 ...

vis[now][num] = true; // 算完立刻打上标记
return dp[now][num] = res;
```

---

## 坑点二：记忆化检查的位置不对（代码臃肿且低效）

**❌ 你的写法：**
在 `for` 循环内部，每次准备调用 DFS 之前，先探头探脑地检查一下 `dp` 数组。
```cpp
for (int i = 0; i < n; i++) {
    if (dp[next_state] == 0) res += dfs(next_state) * p[i];
    else res += dp[next_state] * p[i];
}
```

**💥 为什么不好？**
代码极其冗长，且如果有多个地方需要调用同一个状态，你需要在每个调用的地方都写一遍 `if-else`，极易漏写。

**✅ 优化方案：统一拦截（Lazy 思想）**
不要在调用前检查，而是**大胆地调用**，把检查的工作统一交给 DFS 函数的第一行！
```cpp
double dfs(int now, int num) {
    if (到达终点) return 0.0;
    
    // 统一拦截：进门第一件事先查字典！
    if (vis[now][num]) return dp[now][num]; 
    
    // ...
    for (int i = 0; i < n; i++) {
        res += p[i] * dfs(next_state); // 直接无脑调用，极其清爽
    }
}
```

---

## 坑点三：在 DFS 核心循环中使用 $$O(N)$$ 的自定义函数

**❌ 你的写法：**
写了一个 `while` 循环来统计二进制中有几个 1。
```cpp
int count(int i) {
    int num = 0;
    while (i) {
        if (i & 1) num++;
        i >>= 1;
    }
    return num;
}
```

**💥 为什么会 TLE？**
DFS 的调用次数可能是百万级别的。在这个百万级别的函数里，每次还要跑一个 $$O(N)$$ 的 `while` 循环，时间复杂度直接多乘了一个 $$N$$，常数爆炸。

**✅ 优化方案：使用底层硬件指令 `__builtin_popcount`**
C++ GCC 编译器提供了一个极速内置函数，它是直接调用 CPU 硬件指令来数 1 的，时间复杂度是严格的 $$O(1)$$，这是**状态压缩 DP 的标配**！
```cpp
int c = __builtin_popcount(now); // 极速获取二进制中 1 的个数
```

---

## 坑点四：频繁调用微小函数带来的压栈开销

**❌ 你的写法：**
写了一个 `haven` 函数来判断第 `i` 位是否为 1。
```cpp
bool haven(int i, int now) {
    return (now >> i) & 1;
}
// 在循环里：if (haven(i, now)) ...
```

**💥 为什么不好？**
虽然逻辑很清晰，但在 C++ 中，每次函数调用都需要保存现场、压栈、出栈。在 DFS 的最内层循环做这种事，会带来不必要的常数时间消耗。

**✅ 优化方案：直接内联位运算**
对于这种极其简单的位运算，直接写在 `if` 里，或者使用宏定义/`inline` 关键字。
```cpp
// 直接判断 now 的第 i 位是否为 1
if (now & (1 << i)) { ... } 
```

---

## 🏆 终极总结：记忆化搜索的“八股文”标准模板

以后写任何记忆化搜索，严格按照这个顺序写，保证既不会死循环，也不会 TLE：

```cpp
bool vis[MAX_STATE]; // 1. 必须有专门的 vis 数组
double dp[MAX_STATE];

double dfs(int state) {
    // 第一步：判边界（终点或死胡同）
    if (is_target(state)) return 0.0; 
    
    // 第二步：查备忘录（统一拦截）
    if (vis[state]) return dp[state]; 
    
    // 第三步：推导计算
    double res = 1.0; 
    for (int i = 0; i < n; i++) {
        res += p[i] * dfs(next_state); // 大胆调用，无需在外部检查
    }
    
    // 第四步：存备忘录，并返回
    vis[state] = true; 
    return dp[state] = res; 
}
```
