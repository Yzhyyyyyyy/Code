# 高斯消元法（求矩阵秩）C++ 代码模拟指南

在代码实现中，高斯消元就是一个**带有特定边界控制的三重嵌套循环**。
假设我们有一个二维数组 `vector<vector<double>> a`，大小为 $$m$$ 行 $$n$$ 列。

## 1. 核心变量定义
*   `m`: 矩阵的总行数（例如：方程式的个数）。
*   `n`: 矩阵的总列数（例如：变量的个数）。
*   `col`: **外层主循环指针**。表示当前正在尝试消元的列，从 `0` 遍历到 `n-1`。
*   `rank`: **当前的主元行指针**。表示我们已经成功找到了几个主元（也就是当前的秩）。初始为 `0`。

## 2. 代码模拟的 4 个标准动作
对于每一列 `col`，我们在大循环 `for (int col = 0; col < n; ++col)` 中严格执行以下 4 步：

### 动作一：向下寻找最大主元 (Find Pivot)
在第 `col` 列中，从第 `rank` 行开始一直到最后一行（第 `m-1` 行），找绝对值最大的数所在的行号 `pivot`。
```cpp
int pivot = rank;
for (int i = rank + 1; i < m; ++i) {
    if (abs(a[i][col]) > abs(a[pivot][col])) {
        pivot = i;
    }
}
```

### 动作二：判零与跳过 (Check Zero)
找到最大值后，必须判断它是不是 0。由于是浮点数，不能用 `== 0`，必须引入极小值 `EPS`（通常设为 $$1e-7$$）。
如果最大值都接近 0，说明这一列废了，直接 `continue` 去看下一列（注意此时 `rank` **不增加**）。
```cpp
if (abs(a[pivot][col]) < EPS) {
    continue; 
}
```

### 动作三：交换行 (Swap)
把找到的 `pivot` 行和当前的 `rank` 行交换。C++ 标准库的 `swap` 可以直接交换 `vector` 的一整行，非常高效。
```cpp
if (pivot != rank) {
    swap(a[rank], a[pivot]);
}
```

### 动作四：向下消元 (Eliminate)
这是最核心的计算部分。我们要把第 `rank` 行**正下方**的所有行（从 `rank + 1` 到 `m-1`），在第 `col` 列的位置变成 0。
对于下方的每一行 `i`，计算倍数 `factor`，然后把第 `i` 行的每一列都减去 `factor * a[rank][j]`。
```cpp
for (int i = rank + 1; i < m; ++i) {
    double factor = a[i][col] / a[rank][col];
    // 注意：为了优化常数时间，内层循环可以直接从 col 开始，
    // 因为 col 左边的数本来就已经是 0 了，没必要再减。
    for (int j = col; j < n; ++j) {
        a[i][j] -= factor * a[rank][j];
    }
}
```
消元完成后，说明我们在这一列成功建立了一个主元，此时将 `rank++`。

---

## 3. 完整 C++ 模板代码
把上面的 4 个动作拼起来，就是最标准、最不容易写出 BUG 的模板：

```cpp
#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>

using namespace std;

const double EPS = 1e-7;

int getMatrixRank(vector<vector<double>>& a) {
    if (a.empty() || a[0].empty()) return 0;
    
    int m = a.size();
    int n = a[0].size();
    int rank = 0; // 既是当前要放置主元的行号，也是最终的秩

    // 外层循环：按列推进
    for (int col = 0; col < n && rank < m; ++col) {
        
        // 1. 找主元
        int pivot = rank;
        for (int i = rank + 1; i < m; ++i) {
            if (abs(a[i][col]) > abs(a[pivot][col])) {
                pivot = i;
            }
        }

        // 2. 判零跳过
        if (abs(a[pivot][col]) < EPS) {
            continue;
        }

        // 3. 交换行
        if (pivot != rank) {
            swap(a[rank], a[pivot]);
        }

        // 4. 消元
        for (int i = rank + 1; i < m; ++i) {
            double factor = a[i][col] / a[rank][col];
            for (int j = col; j < n; ++j) {
                a[i][j] -= factor * a[rank][j];
            }
        }
        
        // 成功处理一列，秩 + 1
        rank++;
    }

    return rank;
}
```

## 4. 机试避坑指南
1. **原矩阵会被破坏**：上面的代码直接在传入的 `a` 上修改（引用传递 `&`）。如果后续还需要原矩阵，记得传入前先 `copy` 一份。
2. **为什么循环条件有 `rank < m`？** 因为如果行数少于列数（比如 3 个方程 5 个变量），最多只能找到 3 个主元，`rank` 达到 `m` 后就没行可换了，直接结束即可。
3. **时间复杂度**：$$O(m \times n^2)$$。对于 CSP Q3 的数据规模（通常 $$m, n \le 100$$），这个复杂度跑起来连 1 毫秒都用不到，绝对不会 TLE。
