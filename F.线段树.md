# 🌲 线段树 (Segment Tree) 极简入门指南

在 CSP 考试中，当你遇到需要频繁进行**区间修改**和**区间查询**的题目，且数据范围 $$N \le 10^5$$ 时，线段树就是你的不二之选。

## 1. 核心思想：区间“外包”

线段树本质上是一棵**二叉树**。它的特殊之处在于：**每个节点不存储单个元素，而是存储一个区间的信息（例如区间和、最大值等）。**

*   **根节点**：管理整个数组区间 $$[1, N]$$。
*   **叶子节点**：管理单个元素 $$[i, i]$$。
*   **父子关系**：如果一个节点管理区间 $$[L, R]$$，它的左儿子管理 $$[L, mid]$$，右儿子管理 $$[mid + 1, R]$$，其中 $$mid = \lfloor(L+R)/2\rfloor$$。

**⚠️ 避坑指南**：线段树的数组空间**必须开到原数组大小的 4 倍**（即 `MAXN * 4`），否则必定越界（RE/SegFault）。

---

## 2. 灵魂机制：懒标记 (Lazy Tag)

为什么线段树能把时间复杂度从 $$O(N)$$ 降到 $$O(\log N)$$？全靠“懒标记”。

**通俗理解**：
假设你是大老板（根节点），要给某个部门（某个区间）的所有员工发奖金（区间加法）。
*   **暴力做法**：你亲自跑去给每个员工发钱（遍历到底层叶子节点）。太慢！
*   **懒标记做法**：你把钱打给部门经理（区间节点），并在他头上贴个“懒标记”（记录：手下每人加 $$k$$ 元）。经理把钱收下，更新自己部门的总业绩，但**不立刻把钱发给底层员工**。
*   **什么时候发？** 只有当大老板要查某个具体员工的业绩时，经理才被迫把“懒标记”往下传给小主管。这个“往下传”的动作就叫 **`pushdown`**。

---

## 3. 五大核心函数

掌握这五个函数，你就掌握了线段树的骨架。

### ① `pushup(u)`：向上更新
儿子节点的信息发生变化时，更新父亲节点。
```cpp
void pushup(int u) {
    tr[u].sum = tr[u << 1].sum + tr[u << 1 | 1].sum;
}
```

### ② `build(u, l, r)`：建树（初始化）
给每个节点分配好它要管辖的区间 $$[l, r]$$，并把初始的 `sum` 算出来。
```cpp
void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r, tr[u].add = 0;
    if (l == r) {
        tr[u].sum = w[l]; // w 是存原始数据的数组
        return;
    }
    int mid = (l + r) >> 1;
    build(u << 1, l, mid);
    build(u << 1 | 1, mid + 1, r);
    pushup(u); // 儿子建好了，更新自己
}
```

### ③ `pushdown(u)`：向下传递（最重要！）
把父亲兜里的“懒标记”发给两个儿子。
```cpp
void pushdown(int u) {
    if (tr[u].add) { // 如果有懒标记
        // 1. 更新左儿子的值 = 标记值 * 左儿子区间长度
        tr[u << 1].sum += tr[u].add * (tr[u << 1].r - tr[u << 1].l + 1);
        tr[u << 1].add += tr[u].add; // 标记传给左儿子
        
        // 2. 更新右儿子的值 = 标记值 * 右儿子区间长度
        tr[u << 1 | 1].sum += tr[u].add * (tr[u << 1 | 1].r - tr[u << 1 | 1].l + 1);
        tr[u << 1 | 1].add += tr[u].add; // 标记传给右儿子
        
        // 3. 清空父亲的标记
        tr[u].add = 0;
    }
}
```

### ④ `modify(u, l, r, d)`：区间修改
给区间 $$[l, r]$$ 加上 $$d$$。
```cpp
void modify(int u, int l, int r, long long d) {
    // 1. 如果当前节点区间被完全包含，直接“偷懒”打标记
    if (tr[u].l >= l && tr[u].r <= r) {
        tr[u].sum += d * (tr[u].r - tr[u].l + 1);
        tr[u].add += d;
        return;
    }
    
    // 2. 如果没有完全包含，必须先下传标记！
    pushdown(u);
    
    // 3. 递归修改左右子树
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u << 1, l, r, d);
    if (r > mid)  modify(u << 1 | 1, l, r, d);
    
    // 4. 儿子改完了，更新自己
    pushup(u);
}
```

### ⑤ `query(u, l, r)`：区间查询
查询区间 $$[l, r]$$ 的和。
```cpp
long long query(int u, int l, int r) {
    // 1. 完全包含，直接返回
    if (tr[u].l >= l && tr[u].r <= r) return tr[u].sum;
    
    // 2. 往下走之前，必须下传标记！
    pushdown(u);
    
    // 3. 收集左右子树的答案
    long long res = 0;
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) res += query(u << 1, l, r);
    if (r > mid)  res += query(u << 1 | 1, l, r);
    
    return res;
}
```

---

## 4. 考场实战 CheckList

在 CSP 考场上写线段树，请务必检查以下几点：
- [ ] 数组大小是否开了 `MAXN * 4`？
- [ ] 区间和 `sum` 和懒标记 `add` 是否使用了 `long long`？（CSP 极爱考 `int` 溢出）
- [ ] `modify` 和 `query` 中，只要没有被完全包含（需要往下递归），是否**第一时间调用了 `pushdown(u)`**？
- [ ] `pushdown` 中，儿子增加的值是否乘以了**区间的长度**？
- [ ] `main` 函数里调用 `build`、`modify`、`query` 时，第一个参数 `u` 是否**永远传 `1`**？

---

## 5. 完整实战模板 (Copy-Paste Ready)

这是一个支持“区间加法”和“区间求和”的完整线段树模板，可以直接用于洛谷 P3372 等经典模板题。

```cpp
#include <iostream>
#include <cstdio>

using namespace std;

const int N = 100010; // 根据题目修改大小

int n, m;
int w[N];

struct Node {
    int l, r;
    long long sum;
    long long add;
} tr[N * 4]; // 必须开 4 倍空间！

void pushup(int u) {
    tr[u].sum = tr[u << 1].sum + tr[u << 1 | 1].sum;
}

void pushdown(int u) {
    if (tr[u].add) {
        tr[u << 1].sum += tr[u].add * (tr[u << 1].r - tr[u << 1].l + 1);
        tr[u << 1].add += tr[u].add;
        tr[u << 1 | 1].sum += tr[u].add * (tr[u << 1 | 1].r - tr[u << 1 | 1].l + 1);
        tr[u << 1 | 1].add += tr[u].add;
        tr[u].add = 0;
    }
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r, tr[u].add = 0;
    if (l == r) {
        tr[u].sum = w[l];
        return;
    }
    int mid = (l + r) >> 1;
    build(u << 1, l, mid);
    build(u << 1 | 1, mid + 1, r);
    pushup(u);
}

void modify(int u, int l, int r, long long d) {
    if (tr[u].l >= l && tr[u].r <= r) {
        tr[u].sum += d * (tr[u].r - tr[u].l + 1);
        tr[u].add += d;
        return;
    }
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u << 1, l, r, d);
    if (r > mid)  modify(u << 1 | 1, l, r, d);
    pushup(u);
}

long long query(int u, int l, int r) {
    if (tr[u].l >= l && tr[u].r <= r) return tr[u].sum;
    pushdown(u);
    long long res = 0;
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) res += query(u << 1, l, r);
    if (r > mid)  res += query(u << 1 | 1, l, r);
    return res;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i ++ ) scanf("%d", &w[i]);
    
    build(1, 1, n); // 初始化建树
    
    while (m -- ) {
        int op, l, r;
        long long d;
        scanf("%d%d%d", &op, &l, &r);
        if (op == 1) {
            scanf("%lld", &d);
            modify(1, l, r, d); // 区间加法
        } else {
            printf("%lld\n", query(1, l, r)); // 区间查询
        }
    }
    return 0;
}
```
