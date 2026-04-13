# C++ STL 实战笔记与进阶避坑指南

## 🌟 第一部分：STL 常用容器笔记（原版）

```cpp
// ==================== C++ STL 常用容器笔记 ====================

#include <queue>
#include <stack>
#include <vector>
#include <set>
#include <map>
#include <algorithm>
using namespace std;

// ==================== 1. queue（队列）====================
// 数据结构：先进先出（FIFO），只能从队尾插入、队首删除
// 应用场景：BFS、任务调度、层序遍历

// 定义：
queue<int> q;           // 存储 int 类型
queue<string> qs;       // 存储 string 类型

// 常用函数：
q.push(val);            // 队尾入队，O(1)
q.pop();                // 队首出队，无返回值！O(1)
q.front();              // 返回队首元素（不删除），O(1)
q.back();               // 返回队尾元素（不删除），O(1)
q.empty();              // 判断是否为空，返回 bool，O(1)
q.size();               // 返回元素个数，O(1)

// 注意：
// 1. pop() 无返回值，需要先 front() 获取值，再 pop() 删除
// 2. 访问队首前必须判断 !q.empty()，否则未定义行为
// 3. queue 不支持遍历，只能通过 pop() 逐个取出

// 示例：BFS 模板
queue<int> q;
q.push(start);
visited[start] = true;
while (!q.empty()) {
    int cur = q.front();  // 获取队首
    q.pop();              // 删除队首
    // 处理 cur 的邻接节点
    for (int next : neighbors) {
        if (!visited[next]) {
            visited[next] = true;
            q.push(next);
        }
    }
}


// ==================== 2. priority_queue（优先队列）====================
// 数据结构：堆（Heap），默认大顶堆（每次弹出最大元素）
// 应用场景：Dijkstra、贪心算法、Top K 问题、带权 BFS

// 定义：
priority_queue<int> pq;                          // 大顶堆（默认）
priority_queue<int, vector<int>, greater<int>> pq_min;  // 小顶堆

// 常用函数：
pq.push(val);           // 插入元素，O(log n)
pq.pop();               // 删除堆顶元素，无返回值！O(log n)
pq.top();               // 返回堆顶元素（不删除），O(1)
pq.empty();             // 判断是否为空，O(1)
pq.size();              // 返回元素个数，O(1)

// 自定义排序（重载运算符）：
struct Node {
    int x, y, cost;
    // 重载 < 运算符（必须加 const）
    bool operator < (const Node& b) const {
        return cost > b.cost;  // 小顶堆：cost 小的优先（注意是 >）
    }
};
priority_queue<Node> pq;  // 自动按 cost 从小到大弹出

// 注意：
// 1. 默认大顶堆，重载 < 时写 > 得到小顶堆（反直觉！）
// 2. 重载函数必须加 const，否则编译错误
// 3. 不支持遍历，只能通过 pop() 逐个取出

// 示例：Dijkstra 最短路
priority_queue<Node> pq;
pq.push({start_x, start_y, 0});
while (!pq.empty()) {
    Node cur = pq.top();  // 取出花费最小的节点
    pq.pop();
    if (visited[cur.x][cur.y]) continue;
    visited[cur.x][cur.y] = true;
    // 更新邻接节点
    for (int i = 0; i < 4; i++) {
        int nx = cur.x + dx[i];
        int ny = cur.y + dy[i];
        pq.push({nx, ny, cur.cost + weight});
    }
}


// ==================== 3. stack（栈）====================
// 数据结构：后进先出（LIFO），只能从栈顶插入和删除
// 应用场景：DFS、括号匹配、表达式求值、单调栈

// 定义：
stack<int> st;

// 常用函数：
st.push(val);           // 入栈，O(1)
st.pop();               // 出栈，无返回值！O(1)
st.top();               // 返回栈顶元素（不删除），O(1)
st.empty();             // 判断是否为空，O(1)
st.size();              // 返回元素个数，O(1)

// 注意：
// 1. pop() 无返回值，需要先 top() 再 pop()
// 2. 访问栈顶前必须判断 !st.empty()
// 3. 不支持遍历

// 示例：DFS 非递归实现
stack<int> st;
st.push(start);
while (!st.empty()) {
    int cur = st.top();
    st.pop();
    if (visited[cur]) continue;
    visited[cur] = true;
    // 将邻接节点入栈
    for (int next : neighbors) {
        st.push(next);
    }
}


// ==================== 4. vector（动态数组）====================
// 数据结构：可变长数组，支持随机访问
// 应用场景：存储数据、邻接表、动态规划

// 定义：
vector<int> v;                  // 空数组
vector<int> v(10);              // 10 个元素，初始化为 0
vector<int> v(10, 5);           // 10 个元素，初始化为 5
vector<int> v = {1, 2, 3};      // 初始化列表

// 常用函数：
v.push_back(val);       // 尾部插入，O(1)
v.pop_back();           // 删除尾部元素，O(1)
v[i];                   // 访问第 i 个元素，O(1)
v.front();              // 返回第一个元素，O(1)
v.back();               // 返回最后一个元素，O(1)
v.size();               // 返回元素个数，O(1)
v.empty();              // 判断是否为空，O(1)
v.clear();              // 清空所有元素，O(n)
v.resize(n);            // 调整大小为 n，O(n)
v.insert(it, val);      // 在迭代器 it 位置插入 val，O(n)
v.erase(it);            // 删除迭代器 it 位置的元素，O(n)

// 遍历：
for (int i = 0; i < v.size(); i++) {
    cout << v[i] << " ";
}
for (int x : v) {       // 范围 for（推荐）
    cout << x << " ";
}

// 排序：
sort(v.begin(), v.end());           // 升序
sort(v.begin(), v.end(), greater<int>());  // 降序

// 注意：
// 1. 下标访问 v[i] 不检查越界，v.at(i) 会检查（慢）
// 2. size() 返回 size_t（无符号），与 int 比较时注意类型转换


// ==================== 5. set（集合）====================
// 数据结构：红黑树，自动排序、自动去重
// 应用场景：判重、维护有序集合、范围查询

// 定义：
set<int> s;

// 常用函数：
s.insert(val);          // 插入元素，O(log n)
s.erase(val);           // 删除元素，O(log n)
s.find(val);            // 查找元素，返回迭代器，O(log n)
s.count(val);           // 统计个数（0 或 1），O(log n)
s.size();               // 返回元素个数，O(1)
s.empty();              // 判断是否为空，O(1)
s.clear();              // 清空所有元素，O(n)
s.begin();              // 返回第一个元素的迭代器，O(1)
s.end();                // 返回最后一个元素之后的迭代器，O(1)
s.lower_bound(val);     // 第一个 >= val 的元素，O(log n)
s.upper_bound(val);     // 第一个 > val 的元素，O(log n)

// 判断元素是否存在：
if (s.find(val) != s.end()) { /* 存在 */ }
if (s.count(val)) { /* 存在（更简洁）*/ }

// 遍历（自动排序）：
for (int x : s) {
    cout << x << " ";
}

// 示例：BFS 判重
set<string> visited;
if (visited.find(state) == visited.end()) {  // 未访问过
    visited.insert(state);
    q.push(state);
}

// 注意：
// 1. 自动排序，插入/查找/删除都是 O(log n)
// 2. 不能通过下标访问，只能通过迭代器
// 3. 自动去重，重复插入无效


// ==================== 6. unordered_set（无序集合）====================
// 数据结构：哈希表，不排序、自动去重
// 应用场景：快速判重（比 set 快）

// 定义：
unordered_set<int> us;

// 常用函数（与 set 相同）：
us.insert(val);         // 插入，O(1)
us.erase(val);          // 删除，O(1)
us.find(val);           // 查找，O(1)
us.count(val);          // 统计，O(1)
us.size();              // 大小，O(1)
us.empty();             // 判空，O(1)

// 示例：快速判重
unordered_set<string> visited;
if (visited.find(state) == visited.end()) {
    visited.insert(state);
}

// 注意：
// 1. 比 set 快（O(1) vs O(log n)），但不排序
// 2. 不支持 lower_bound/upper_bound


// ==================== 7. map（映射）====================
// 数据结构：红黑树，存储键值对，按键排序
// 应用场景：统计频率、建立映射关系

// 定义：
map<string, int> m;

// 常用函数：
m[key] = value;         // 插入/修改，O(log n)
m.erase(key);           // 删除键，O(log n)
m.find(key);            // 查找键，返回迭代器，O(log n)
m.count(key);           // 判断键是否存在（0 或 1），O(log n)
m.size();               // 返回键值对个数，O(1)
m.empty();              // 判断是否为空，O(1)

// 遍历（按键排序）：
for (auto& p : m) {
    cout << p.first << ": " << p.second << endl;  // first 是键，second 是值
}

// 示例：统计字符串出现次数
map<string, int> cnt;
for (string s : words) {
    cnt[s]++;  // 自动初始化为 0
}

// 注意：
// 1. m[key] 若 key 不存在，会自动创建并初始化为 0
// 2. 按键排序，操作都是 O(log n)


// ==================== 8. unordered_map（无序映射）====================
// 数据结构：哈希表，存储键值对，不排序
// 应用场景：快速查找映射关系（比 map 快）

// 定义：
unordered_map<string, int> um;

// 常用函数（与 map 相同）：
um[key] = value;        // 插入/修改，O(1)
um.erase(key);          // 删除，O(1)
um.find(key);           // 查找，O(1)
um.count(key);          // 判断存在，O(1)

// 示例：快速统计
unordered_map<int, int> cnt;
for (int x : arr) {
    cnt[x]++;
}

// 注意：
// 1. 比 map 快，但不排序
// 2. 键必须可哈希（int、string 可以，自定义结构体需要写哈希函数）


// ==================== 9. pair（键值对）====================
// 数据结构：存储两个元素的结构体
// 应用场景：返回多个值、存储坐标、Dijkstra 中存储 (距离, 节点)

// 定义：
pair<int, int> p;
pair<int, string> p2 = {1, "hello"};
pair<int, int> p3 = make_pair(1, 2);

// 访问：
p.first;                // 第一个元素
p.second;               // 第二个元素

// 比较：
// 先比较 first，相同则比较 second
pair<int, int> a = {1, 2};
pair<int, int> b = {1, 3};
if (a < b) { /* a.first == b.first，比较 second，2 < 3 */ }

// 示例：Dijkstra 中存储 (距离, 节点)
priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
pq.push({0, start});  // {距离, 节点}
while (!pq.empty()) {
    auto [dist, node] = pq.top();  // C++17 结构化绑定
    pq.pop();
}


// ==================== 10. algorithm（算法库）====================
// 常用函数：

// 排序：
sort(v.begin(), v.end());                   // 升序，O(n log n)
sort(v.begin(), v.end(), greater<int>());   // 降序

// 查找：
find(v.begin(), v.end(), val);              // 返回迭代器，O(n)
binary_search(v.begin(), v.end(), val);     // 二分查找（需有序），返回 bool，O(log n)
lower_bound(v.begin(), v.end(), val);       // 第一个 >= val 的位置，O(log n)
upper_bound(v.begin(), v.end(), val);       // 第一个 > val 的位置，O(log n)

// 最值：
max(a, b);                                  // 返回较大值
min(a, b);                                  // 返回较小值
*max_element(v.begin(), v.end());           // 返回最大元素，O(n)
*min_element(v.begin(), v.end());           // 返回最小元素，O(n)

// 反转：
reverse(v.begin(), v.end());                // 反转数组，O(n)

// 去重（需先排序）：
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());  // 去重，O(n)

// 填充：
fill(v.begin(), v.end(), val);              // 填充为 val，O(n)


// ==================== 总结对比 ====================
// 容器           底层结构    有序   去重   时间复杂度      应用场景
// queue          队列        ×      ×      O(1)           BFS、任务调度
// priority_queue 堆          ×      ×      O(log n)       Dijkstra、Top K
// stack          栈          ×      ×      O(1)           DFS、括号匹配
// vector         动态数组    ×      ×      O(1) 访问      存储数据
// set            红黑树      √      √      O(log n)       判重、有序集合
// unordered_set  哈希表      ×      √      O(1)           快速判重
// map            红黑树      √      √      O(log n)       有序映射
// unordered_map  哈希表      ×      √      O(1)           快速映射

// 选择建议：
// 1. 需要排序 → set/map
// 2. 只需判重 → unordered_set
// 3. 需要最值 → priority_queue
// 4. 需要顺序处理 → queue（BFS）、stack（DFS）
// 5. 需要随机访问 → vector
```

---

## 💡 第二部分：考场实战踩坑与进阶技巧（新增补充）

### 1. 自定义排序：`sort` vs `set` 的巨大区别
在考场上，给自定义结构体排序是最常见的需求，但 `sort` 和 `set` 的语法要求完全不同，极易踩坑！

*   **对 `sort` 排序（推荐写独立 `cmp` 函数）**
    `sort` 是一个函数，可以直接传入独立的比较函数，语法最简单直观。
    ```cpp
    struct Student { int id, score; };
    // 独立的 cmp 函数
    bool cmp(Student a, Student b) { return a.score < b.score; } 
    
    // 使用：
    sort(arr, arr + n, cmp);
    ```

*   **把结构体塞进 `set` / `map`（推荐在结构体内重载 `<`）**
    `set` 是一个容器模板，**不能直接传函数名**。如果强行写外挂 `cmp`，必须写成复杂的“仿函数”（Functor）。因此，考场上最稳妥的做法是**直接在结构体内部重载 `<` 运算符**。
    ```cpp
    struct Student {
        int id, score;
        // 必须加两个 const！
        bool operator < (const Student& b) const {
            return score < b.score; 
        }
    };
    
    // 使用：
    set<Student> s; // 直接塞，set 会自动调用你写的 < 规则
    ```

### 2. 为什么存坐标用 `set` 而不是速度更快的 `unordered_set`？
从算法逻辑上，单纯存坐标并查找，确实不需要排序，哈希表（`unordered_set`）是理论最优解。但 C++ 有一个底层硬伤：

*   **致命坑点**：C++ 官方**没有为 `pair` 提供哈希函数**！如果你在考场上写下 `unordered_set<pair<int, int>>`，会直接**编译报错（CE）**！
*   **考场最优解**：直接使用 `set<pair<int, int>>`。虽然底层红黑树会多余地排个序，但 C++ 官方已经帮 `pair` 写好了比较规则（先比 first 再比 second），一行代码都不用多写，且 $$O(\log N)$$ 的时间复杂度在 $$N \le 10^5$$ 的数据量下完全不会超时。
*   **极限操作（坐标压缩）**：如果非要用 `unordered_set` 追求极限速度，可以把二维坐标压缩成一个 `long long`：
    ```cpp
    unordered_set<long long> locate;
    long long hash_val = (long long)x * 2000000000LL + y; // 降维打击
    locate.insert(hash_val);
    ```

### 3. `priority_queue` 的重载运算符（反直觉大坑）
普通排序时，`return a < b` 代表从小到大排。但在 `priority_queue`（优先队列）中，逻辑是**反过来**的！

*   **底层逻辑**：优先队列默认是**大顶堆**。它通过你重载的 `<` 符号来判断，**谁在 `<` 的右边（谁更大），谁就浮到堆顶**。
*   **如何写小顶堆（如 Dijkstra 算法）**：为了让 `cost` 最小的节点浮到堆顶，我们必须“欺骗” C++，故意把逻辑写反（用 `>`）。
    ```cpp
    struct Node {
        int id, cost;
        bool operator < (const Node& b) const {
            return cost > b.cost;  // 故意写反！得到小顶堆
        }
    };
    ```

### 4. `map` 与 `set` 的底层灵魂：键（Key）的绝对只读性
*   **`map<Key, Value>` 的本质是“带锁的柜子”**：
    *   **键（Key）**：柜子门上的号码牌。一旦贴上去，**绝对不能修改**（底层强制加了 `const`）。改了号码牌，红黑树就全乱了。
    *   **值（Value）**：柜子里面的东西。可以随意修改（如 `m["alice"]++`）。
*   **`set<Key>` 的本质是“刻在石头上的名字”**：
    *   它**只有键（Key），没有值（Value）**。
    *   一旦 `insert` 进去，元素就是**只读（const）**的。绝对不能通过迭代器修改它的值。如果非要改，只能 `erase` 删掉旧的，再 `insert` 新的。
*   **考场口诀**：
    *   需要**“查找 + 频繁修改数据”**（如统计频次、更新最早时间） $\rightarrow$ 无脑选 `map` / `unordered_map`。
    *   需要**“自动排序 + 自动去重 + 只看不改”** $\rightarrow$ 无脑选 `set`。

### 5. 到底什么时候必须重载 `<` 运算符？
在 CSP 考场上，只有以下 3 个场景**绝对逃不掉**重载 `<`：
1.  **`priority_queue`（优先队列/堆）**：只要塞入自定义 `struct`，必写重载，否则 CE。
2.  **把 `struct` 作为 `map` 或 `set` 的键（Key）**：比如用二维坐标 `struct Point` 作为 `map` 的键统计频次，必须重载 `<` 让红黑树知道怎么建树。
3.  **`set` 的动态维护与二分查找**：当需要用 `set` 动态插入/删除自定义结构体，并使用 `lower_bound` 快速查找时。

### 6. 迭代器（Iterator）的正确打开方式与 `lower_bound` 神技
`lower_bound` 返回的是迭代器（可视为指向红黑树节点的“高级指针”）。考场上拿到迭代器后，必须熟练掌握以下“三把钥匙”：
1.  **验明正身（防 RE 神器）**：拿到迭代器第一件事，永远是判断 `if (it == s.end())`。如果不检查直接用，一旦没找到就会越界 RE。
2.  **提取数据（解引用）**：确认安全后，用 `*it`（基本类型）或 `it->name`（结构体）获取里面的值。
3.  **前后移动（考场高阶技巧）**：
    *   `lower_bound(val)` 找的是**第一个 $$\ge val$$ 的元素**。
    *   **怎么找最后一个 $$< val$$ 的元素？** 先用 `lower_bound` 找到位置，判断 `if (it != s.begin())` 后，直接 **`it--`** 往回退一步即可！

### 7. 容器的逆序操作：从大到小排序与反向遍历（神技）
考场上经常遇到需要“从大到小”处理数据的场景，STL 提供了两种极其优雅的解决方案：

*   **方法一：从根源上倒排（直接使用 `greater`）**
    不仅 `int` 可以用 `greater<int>`，`pair` 也完全可以直接套用！C++ 官方已经为 `pair` 写好了大于号 `>` 的比较规则（先比 `first`，相同再比 `second`）。
    ```cpp
    // 定义一个从大到小排序的 set，里面存 pair
    set<pair<int, int>, greater<pair<int, int>>> s;
    s.insert({1, 5});
    s.insert({3, 8});
    s.insert({3, 2});
    // 遍历输出顺序：{3, 8} -> {3, 2} -> {1, 5}
    ```
    👉 *适用场景*：核心逻辑就是每次都要取最大值（例如直接用 `s.begin()` 拿最大元素）。

*   **方法二：只在遍历时倒着看（使用反向迭代器 `rbegin` / `rend`）**
    如果你不想改变容器默认从小到大的排序规则（比如还需要用 `lower_bound` 进行二分查找），只是想在输出时从大到小，可以直接使用反向迭代器。
    *   `rbegin()`：指向最后一个元素（Reverse Begin）。
    *   `rend()`：指向第一个元素的前一个位置（Reverse End）。
    ```cpp
    map<int, string> m;
    m[1] = "Alice"; m[5] = "Bob"; m[3] = "Charlie";

    // 注意：反向迭代器往前走依然是 ++，但物理上是往回退！
    for (auto it = m.rbegin(); it != m.rend(); ++it) {
        cout << it->first << " -> " << it->second << endl;
    }
    // 输出顺序：5 -> 3 -> 1
    ```
    👉 *适用场景*：平时需要从小到大处理或二分查找，仅在特定时刻（如输出答案）需要逆序。
