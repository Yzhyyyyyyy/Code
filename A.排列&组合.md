# 排列组合核心算法笔记

在算法竞赛中，求组合数 $$ \binom{n}{m} $$（从 $$n$$ 个物品中选 $$m$$ 个的方案数）主要分为两大类场景。针对不同的场景，必须使用完全不同的算法，否则极易导致超时 (TLE) 或死循环。

---

## 场景一：求“单个”巨大的组合数（质因数分解法）

**适用场景**：$$n, m$$ 比较大，只需要求一次或少数几次组合数，且可能需要对某个数取模，或者直接高精度乘法求真实值。
**核心公式**：$$ \binom{n}{m} = \frac{n!}{m!(n-m)!} $$

因为阶乘增长极快，直接算必然溢出。我们的策略是：**将分子和分母的所有数进行质因数分解，然后将底数相同的指数相减，最后再把剩下的质因数乘起来。**

### 极速质因数分解（最小质因子筛 + DP 递推）

这是处理批量数字质因数分解的最优解法，时间复杂度接近 $$O(N)$$。

**完整流程拆解：**
1. **找最小质因子（预处理）**：利用埃氏筛的思想，开一个数组 `min_prime[i]`，记录每个数 $$i$$ 的最小质因子。
2. **状态转移（DP 思想）**：
   - 如果 $$i$$ 是素数，它的质因数就是它自己。
   - 如果 $$i$$ 是合数，设它的最小质因子为 $$p$$，那么 $$i$$ 的质因数分解结果，等于 **$$i/p$$ 的分解结果，再加上一个 $$p$$**。
   - **注意合并逻辑**：因为 $$p$$ 是最小质因子，所以它一定排在最前面。如果前驱状态的第一个质因子也是 $$p$$，直接指数加 1；否则，将 $$p$$ 插入到最前面。

**标准模板代码：**

```cpp
#include <iostream>
#include <vector>
using namespace std;

const int MAXN = 2010;
int min_prime[MAXN]; 
vector<pair<int, int>> yinshu[MAXN]; // 存质因数分解结果：{底数, 指数}

void init_factors() {
    // 1. 筛法求最小质因子
    for (int i = 2; i < MAXN; i++) min_prime[i] = i;
    for (int i = 2; i < MAXN; i++) {
        if (min_prime[i] == i) {
            int t = 2;
            while (i * t < MAXN) {
                if (min_prime[i * t] == i * t) {
                    min_prime[i * t] = i;
                }
                t++;
            }
        }
    }

    // 2. DP 递推求所有数的质因数分解
    for (int i = 2; i < MAXN; i++) {
        if (min_prime[i] == i) {
            // 素数直接存自己
            yinshu[i].push_back({i, 1});
        } else {
            int p = min_prime[i];
            // 抄前驱状态
            yinshu[i] = yinshu[i / p];
            
            // 严谨的合并逻辑
            if (!yinshu[i].empty() && yinshu[i][0].first == p) {
                yinshu[i][0].second++; // 底数相同，指数 +1
            } else {
                yinshu[i].insert(yinshu[i].begin(), {p, 1}); // 底数不同，插到最前面
            }
        }
    }
}
```

---

## 场景二：求“大量”组合数（杨辉三角递推法）

**适用场景**：$$n, m$$ 范围不大（通常 $$\le 2000$$），但是询问次数极多（比如 $$10^4$$ 次查询）。
**核心公式**：$$ \binom{i}{j} = \binom{i-1}{j} + \binom{i-1}{j-1} $$

这种场景下，如果每次都去分解质因数，时间复杂度会爆炸。最优策略是**在处理询问之前，利用杨辉三角的性质，把所有的组合数全部预处理出来**（通常伴随取模操作）。

**核心思想：**
- 组合数的本质就是杨辉三角。
- 第 $$i$$ 行第 $$j$$ 列的数，等于它正上方和左上方的两个数之和。
- 边界条件：$$ \binom{i}{0} = 1 $$（从 $$i$$ 个里选 0 个，只有 1 种方案）。

**标准模板代码：**

```cpp
#include <iostream>
using namespace std;

const int MAXN = 2005;
int c[MAXN][MAXN]; // c[i][j] 表示从 i 个选 j 个的组合数

void init_combinations(int k) {
    // 预处理组合数，并对 k 取模
    for (int i = 0; i < MAXN; i++) {
        c[i][0] = 1 % k; // 边界条件
    }

    for (int i = 1; i < MAXN; i++) {
        for (int j = 1; j <= i; j++) {
            // 递推公式
            c[i][j] = (c[i - 1][j] + c[i - 1][j - 1]) % k;
        }
    }
}
```

**总结对比：**
- 如果题目问：**“求 $$ \binom{1000}{500} $$ 的精确值或质因子”** $$\rightarrow$$ 用**场景一**。
- 如果题目问：**“给定 10000 组 $$n, m$$，每次求 $$ \binom{n}{m} \pmod k $$”** $$\rightarrow$$ 用**场景二**。
