# C++ STL 实战笔记与进阶避坑指南

## 🌟 第一部分：STL 常用容器笔记

```cpp
// ==================== C++ STL 常用容器笔记 ====================

#include <queue>
#include <stack>
#include <vector>
#include <list>
#include <set>
#include <map>
#include <unordered_set>
#include <unordered_map>
#include <algorithm>
using namespace std;

// ==================== 1. queue（队列）====================
// 数据结构：先进先出（FIFO），只能从队尾插入、队首删除
// 应用场景：BFS、任务调度、层序遍历

queue<int> q;

q.push(val);            // 队尾入队，O(1)
q.pop();                // 队首出队，无返回值！O(1)
q.front();              // 返回队首元素，O(1)
q.back();               // 返回队尾元素，O(1)
q.empty();              // 判断是否为空，O(1)
q.size();               // 返回元素个数，O(1)

// 注意：
// 1. pop() 无返回值，需要先 front() 获取值，再 pop() 删除
// 2. 访问队首前必须判断 !q.empty()
// 3. queue 不支持遍历，只能通过 pop() 逐个取出


// ==================== 2. priority_queue（优先队列）====================
// 数据结构：堆，默认大顶堆
// 应用场景：Dijkstra、贪心、Top K

priority_queue<int> pq; 
priority_queue<int, vector<int>, greater<int>> pq_min;

pq.push(val);           // 插入，O(log n)
pq.pop();               // 删除堆顶，无返回值，O(log n)
pq.top();               // 返回堆顶，O(1)
pq.empty();             // 判空，O(1)
pq.size();              // 大小，O(1)

struct Node {
    int x, y, cost;

    bool operator < (const Node& b) const {
        return cost > b.cost;  // 小顶堆写法：cost 小的优先
    }
};

priority_queue<Node> node_pq;

// 注意：
// 1. 默认大顶堆
// 2. 自定义结构体进 priority_queue，通常要重载 <
// 3. 想让小的先出来，operator< 里常常要反着写


// ==================== 3. stack（栈）====================
// 数据结构：后进先出（LIFO）
// 应用场景：DFS、括号匹配、表达式求值、单调栈

stack<int> st;

st.push(val);           // 入栈，O(1)
st.pop();               // 出栈，无返回值，O(1)
st.top();               // 栈顶，O(1)
st.empty();             // 判空，O(1)
st.size();              // 大小，O(1)

// 注意：
// 1. pop() 无返回值，需要先 top() 再 pop()
// 2. 访问 top() 前必须判断 !st.empty()
// 3. stack 不支持遍历


// ==================== 4. vector（动态数组）====================
// 数据结构：动态数组，支持随机访问
// 应用场景：存储数据、邻接表、动态规划

vector<int> v;
vector<int> v1(10);
vector<int> v2(10, 5);
vector<int> v3 = {1, 2, 3};

v.push_back(val);       // 尾插，均摊 O(1)
v.pop_back();           // 删除尾部，O(1)
v[i];                   // 随机访问，O(1)
v.front();              // 首元素
v.back();               // 尾元素
v.size();               // 大小
v.empty();              // 判空
v.clear();              // 清空
v.resize(n);            // 改变大小
v.insert(it, val);      // 中间插入，O(n)
v.erase(it);            // 中间删除，O(n)

sort(v.begin(), v.end());                    // 升序
sort(v.begin(), v.end(), greater<int>());    // 降序

// 注意：
// 1. v[i] 不检查越界
// 2. v.at(i) 会检查越界，但更慢
// 3. size() 返回 size_t，和 int 比较时注意类型


// ==================== 5. list（双向链表）====================
// 数据结构：双向链表，内存不连续
// 应用场景：频繁中间插入删除，不需要随机访问

list<int> l;

l.push_back(val);       // 尾插，O(1)
l.push_front(val);      // 头插，O(1)
l.pop_back();           // 删除尾部，O(1)
l.pop_front();          // 删除头部，O(1)
l.front();              // 头部元素
l.back();               // 尾部元素
l.empty();              // 判空
l.size();               // 大小
l.insert(it, val);      // 在 it 前插入，O(1)
l.erase(it);            // 删除 it，O(1)，返回下一个迭代器

// 注意：
// 1. list 不能用下标访问
// 2. list 查找慢，find 是 O(n)
// 3. erase 后，被删的迭代器失效


// ==================== 6. set（集合）====================
// 数据结构：红黑树，自动排序，自动去重
// 应用场景：判重、有序集合、范围查询

set<int> s;

s.insert(val);          // 插入，O(log n)
s.erase(val);           // 删除，O(log n)
s.find(val);            // 查找，O(log n)
s.count(val);           // 是否存在，O(log n)
s.lower_bound(val);     // 第一个 >= val
s.upper_bound(val);     // 第一个 > val

// 注意：
// 1. set 自动排序
// 2. set 自动去重
// 3. set 中元素不能直接修改，想改只能 erase 后 insert


// ==================== 7. unordered_set（无序集合）====================
// 数据结构：哈希表，不排序，自动去重
// 应用场景：快速判重

unordered_set<int> us;

us.insert(val);         // 插入，均摊 O(1)
us.erase(val);          // 删除，均摊 O(1)
us.find(val);           // 查找，均摊 O(1)
us.count(val);          // 判断存在，均摊 O(1)

// 注意：
// 1. unordered_set 不排序
// 2. 不支持 lower_bound / upper_bound
// 3. key 必须可以哈希


// ==================== 8. map（映射）====================
// 数据结构：红黑树，按 key 排序
// 应用场景：统计频率、有序映射

map<string, int> mp;

mp[key] = value;        // 插入或修改，O(log n)
mp.erase(key);          // 删除，O(log n)
mp.find(key);           // 查找，O(log n)
mp.count(key);          // 判断存在，O(log n)

// 注意：
// 1. mp[key] 如果 key 不存在，会自动创建
// 2. map 按 key 有序
// 3. key 不能直接修改，value 可以修改


// ==================== 9. unordered_map（无序映射）====================
// 数据结构：哈希表，不按 key 排序
// 应用场景：快速映射、快速计数

unordered_map<string, int> ump;

ump[key] = value;       // 插入或修改，均摊 O(1)
ump.erase(key);         // 删除，均摊 O(1)
ump.find(key);          // 查找，均摊 O(1)
ump.count(key);         // 判断存在，均摊 O(1)

// 注意：
// 1. unordered_map 不排序
// 2. 不支持 lower_bound / upper_bound
// 3. key 必须可以哈希
// 4. int、long long、string 可以直接作为 key
// 5. pair、自定义 struct 通常不能直接作为 key，需要自定义哈希


// ==================== 10. pair（键值对）====================

pair<int, int> p = {1, 2};

p.first;                // 第一个元素
p.second;               // 第二个元素

// pair 默认先比较 first，再比较 second
pair<int, int> a = {1, 2};
pair<int, int> b = {1, 3};

if (a < b) {
    // true
}


// ==================== 11. algorithm（算法库）====================

sort(v.begin(), v.end());                          // 升序
sort(v.begin(), v.end(), greater<int>());          // 降序

find(v.begin(), v.end(), val);                     // 线性查找
binary_search(v.begin(), v.end(), val);            // 二分查找，要求有序
lower_bound(v.begin(), v.end(), val);              // 第一个 >= val
upper_bound(v.begin(), v.end(), val);              // 第一个 > val

max(a, b);
min(a, b);
*max_element(v.begin(), v.end());
*min_element(v.begin(), v.end());

reverse(v.begin(), v.end());

sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());

fill(v.begin(), v.end(), val);
```

---

## 💡 第二部分：考场实战踩坑与进阶技巧

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
    *   `lower_bound(val)` 找的是**第一个 $\ge val$ 的元素**。
    *   **怎么找最后一个 $< val$ 的元素？** 先用 `lower_bound` 找到位置，判断 `if (it != s.begin())` 后，直接 **`it--`** 往回退一步即可！

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

### 8. 迭代器的绝对领域：左闭右开 `[begin(), end())`
在 C++ STL 中，所有容器的迭代器都严格遵循**“左闭右开”**原则：
*   **`begin()`**：精准指向容器的**第 1 个元素**。直接 `*begin()` 拿到的就是首元素。
*   **`end()`**：指向**最后一个元素的下一个位置**（一个虚拟的、越界的位置）。它本身不存储任何有效数据，仅仅作为“遍历结束的标志”。
*   **空容器陷阱**：当容器为空时，`begin()` 会直接等于 `end()`。此时对 `begin()` 解引用会导致程序崩溃。
```text
示例： list<int> l = {10, 20, 30};
      10        20        30       (越界/不存在)
      ↑                             ↑
   l.begin()                     l.end()
```

### 9. 告别又臭又长的迭代器类型：`auto` 与 `decltype` 神法
手写 `list<int>::iterator` 或 `map<string, vector<int>>::iterator` 既浪费时间又容易拼错。
*   **单变量声明：无脑用 `auto`**
    ```cpp
    auto it = l.begin(); 
    auto it2 = find(l.begin(), l.end(), 3);
    ```
*   **定义数组/容器时：用 `decltype` 魔法**
    `decltype` 的意思是 **Declare Type（声明的类型）**。它可以让编译器“照抄”括号里表达式的类型。
    ```cpp
    // 意思是：l.begin() 是啥类型，我的 vector 就存啥类型！
    vector<decltype(l.begin())> pos(100005); 
    ```
    这招在需要开数组存储迭代器时，简直是降维打击！

### 10. 链表（`list`）满分大招：`unordered_map` 缓存迭代器，化 $$O(N^2)$$ 为 $$O(1)$$
`list` 的插入和删除极快（$$O(1)$$），但查找极慢（`find` 是 $$O(N)$$）。如果在 $$N$$ 次循环中每次都用 `find` 找人，总复杂度会飙升到 $$O(N^2)$$ 导致 TLE。

**破局核心**：`list` 插入或删除元素时，**其他元素的迭代器绝对不会失效**（不像 `vector` 会内存大搬家）。
**满分战术**：开一个 `unordered_map`，把每次 `insert` 或 `push` 得到的迭代器直接存起来！下次找人时，直接查表，瞬间实现 $$O(1)$$ 定位！
```cpp
list<int> l;
// 核心魔法：用 unordered_map 专门存每个数字在 list 中的迭代器位置
unordered_map<int, list<int>::iterator> pos;

l.push_front(10);
pos[10] = l.begin(); // 记录 10 的位置

// 删除时 O(1) 秒杀，无需 find：
auto it = pos[10];
l.erase(it);
pos.erase(10);       // 务必同步删除映射！
```

**⚠️ `unordered_map` 的致命限制与避坑**：
1. **绝对无序**：它不按 key 排序，遍历顺序是随机的。如果需要顺序遍历或使用 `lower_bound` 二分查找，**必须换成 `map`**。
2. **Key 必须可哈希**：
   * `int`、`long long`、`string` 等自带哈希，可以直接当 key。
   * **`pair<int, int>` 和自定义 `struct` 默认没有哈希函数**！直接写 `unordered_map<pair<int,int>, int>` 会编译报错（CE）。
   * *考场最优解*：把二维坐标压成一个 `long long`（例如 `x * 2000000000LL + y`）作为 key，或者老老实实用 `map`。
