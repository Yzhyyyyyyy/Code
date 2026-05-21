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

在考场上，给自定义结构体排序是最常见的需求，但 `sort` 和 `set` 的语法要求完全不同，极易踩坑。

#### 对 `sort` 排序：推荐写独立 `cmp` 函数

```cpp
struct Student {
    int id, score;
};

bool cmp(Student a, Student b) {
    return a.score < b.score;
}

sort(arr, arr + n, cmp);
```

#### 把结构体塞进 `set` / `map`：推荐在结构体内重载 `<`

```cpp
struct Student {
    int id, score;

    bool operator < (const Student& b) const {
        return score < b.score;
    }
};

set<Student> s;
```

注意：

```cpp
bool operator < (const Student& b) const
```

后面的两个 `const` 很重要：

- 参数 `const Student& b`：避免拷贝，保证不修改对方；
- 函数末尾 `const`：保证比较时不修改自己。

---

### 2. 为什么存坐标用 `set` 而不是速度更快的 `unordered_set`？

从算法逻辑上看，单纯存坐标并查找，确实不需要排序，哈希表理论上更快。

但 C++ 有一个常见坑：

> C++ 标准库没有给 `pair<int, int>` 默认提供哈希函数。

所以这句在很多环境下会直接编译错误：

```cpp
unordered_set<pair<int, int>> s;
```

考场最稳做法：

```cpp
set<pair<int, int>> s;
```

因为 `pair` 自带比较规则，可以直接放入 `set`。

如果非要使用 `unordered_set`，可以把二维坐标压成一个 `long long`：

```cpp
long long encode(int x, int y) {
    return (long long)x * 2000000000LL + y;
}

unordered_set<long long> locate;
locate.insert(encode(x, y));
```

---

### 3. `priority_queue` 的重载运算符：反直觉大坑

`priority_queue` 默认是大顶堆。

如果想让 `cost` 小的元素先出来，需要反着写：

```cpp
struct Node {
    int id, cost;

    bool operator < (const Node& b) const {
        return cost > b.cost;
    }
};
```

口诀：

```text
priority_queue 想要小的先出，operator< 里面常常写 >
```

---

### 4. `map` 与 `set` 的底层灵魂：Key 的绝对只读性

#### `map<Key, Value>`

`map` 的本质是：

```text
key 是柜子门牌号，value 是柜子里的东西
```

- key 不能修改；
- value 可以修改。

例如：

```cpp
map<string, int> mp;
mp["alice"]++;
```

这里修改的是 value。

#### `set<Key>`

`set` 只有 key，没有 value。

所以元素一旦插入，不能直接修改。

如果非要改：

```cpp
s.erase(old_value);
s.insert(new_value);
```

---

### 5. 到底什么时候必须重载 `<` 运算符？

在 CSP 考场上，下面几个场景很常见：

1. `priority_queue` 存自定义结构体；
2. `set` 存自定义结构体；
3. `map` 的 key 是自定义结构体；
4. 需要让结构体支持排序。

示例：

```cpp
struct Point {
    int x, y;

    bool operator < (const Point& b) const {
        if (x != b.x) return x < b.x;
        return y < b.y;
    }
};
```

---

### 6. 迭代器与 `lower_bound` 神技

`lower_bound(val)` 的含义：

```text
找第一个 >= val 的位置
```

对 `set`：

```cpp
auto it = s.lower_bound(val);
```

拿到迭代器之后，第一件事通常是检查：

```cpp
if (it != s.end()) {
    cout << *it << endl;
}
```

找最后一个 `< val` 的元素：

```cpp
auto it = s.lower_bound(val);

if (it != s.begin()) {
    --it;
    cout << *it << endl;
}
```

注意：

- `end()` 不能解引用；
- `begin()` 再 `--` 会炸；
- 空容器时 `begin() == end()`。

---

### 7. 容器的逆序操作：从大到小排序与反向遍历

#### 方法一：使用 `greater`

```cpp
set<int, greater<int>> s;
```

对 `pair` 也可以：

```cpp
set<pair<int, int>, greater<pair<int, int>>> s;
```

`pair` 的比较规则依旧是：

```text
先比 first，再比 second
```

只不过整体变成从大到小。

#### 方法二：使用反向迭代器

```cpp
map<int, string> mp;

for (auto it = mp.rbegin(); it != mp.rend(); ++it) {
    cout << it->first << " " << it->second << endl;
}
```

注意：

```cpp
++it
```

对反向迭代器来说，逻辑上是往前走，但物理上是从大到小遍历。

---

### 8. 迭代器的绝对领域：左闭右开 `[begin(), end())`

STL 容器统一遵循：

```text
[begin(), end())
```

含义：

- `begin()` 指向第一个元素；
- `end()` 指向最后一个元素的下一个位置；
- `end()` 不能解引用；
- 空容器时 `begin() == end()`。

示例：

```text
list<int> l = {10, 20, 30};

    10        20        30       虚拟位置
    ↑                             ↑
 begin()                         end()
```

---

### 9. 告别又臭又长的迭代器类型：`auto` 与 `decltype`

手写复杂迭代器类型很麻烦：

```cpp
map<string, vector<int>>::iterator it;
```

考场推荐：

```cpp
auto it = mp.begin();
```

如果要定义“存迭代器”的容器，可以用 `decltype`：

```cpp
list<int> l;
vector<decltype(l.begin())> pos(100005);
```

含义：

```text
l.begin() 是什么类型，pos 里就存什么类型
```

但是要注意：

> 如果编号范围很大、不连续，就不要用 vector 存迭代器，要用 unordered_map。

---

### 10. 链表（`list`）满分大招：`unordered_map` 缓存迭代器，化 $$O(N^2)$$ 为均摊 $$O(1)$$

`list` 的插入和删除极快，都是 $$O(1)$$。

但是 `list` 有一个巨大缺点：

> 查找极慢。

比如：

```cpp
auto it = find(l.begin(), l.end(), x);
```

这是线性查找，复杂度是：

$$O(N)$$

如果每次操作都这么找，总复杂度可能变成：

$$O(N^2)$$

直接 TLE。

---

#### 破局核心

`list` 有一个非常重要的性质：

> 插入或删除某个节点时，其他节点的迭代器不会失效。

更准确地说：

- `insert` 不会让已有元素的迭代器失效；
- `erase(it)` 只会让被删除的那个迭代器失效；
- 其他节点的迭代器仍然有效。

所以我们可以用一个映射表记录：

```text
某个元素 x 当前在 list 里的位置
```

如果元素编号很小且连续，可以用 `vector`：

```cpp
vector<list<int>::iterator> pos(max_id + 1);
```

但是如果元素编号很大、不连续，例如题目里内存块编号可能达到：

$$2^{30}$$

那就不能直接拿编号当数组下标。

这时应该使用：

```cpp
unordered_map<int, list<int>::iterator> pos;
```

---

#### 核心写法

```cpp
list<int> l;

// key：元素编号
// value：该元素在 list 中的迭代器
unordered_map<int, list<int>::iterator> pos;
```

含义：

```text
pos[x] = x 在链表中的位置
```

以后想找 `x`，不要再：

```cpp
find(l.begin(), l.end(), x); // O(N)
```

而是：

```cpp
auto it = pos[x]; // 均摊 O(1)
```

---

#### LRU 模板

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int capacity = 3;

    list<int> cache;

    // 记录每个元素在 list 中的位置
    unordered_map<int, list<int>::iterator> pos;

    auto access = [&](int x) {
        // 命中
        if (pos.count(x)) {
            cache.erase(pos[x]);
            cache.push_front(x);
            pos[x] = cache.begin();
        }
        // 未命中
        else {
            if ((int)cache.size() == capacity) {
                int old = cache.back();
                cache.pop_back();
                pos.erase(old);
            }

            cache.push_front(x);
            pos[x] = cache.begin();
        }
    };

    access(1);
    access(2);
    access(3);
    access(1);
    access(4);

    for (int x : cache) {
        cout << x << " ";
    }

    return 0;
}
```

最后链表中：

```text
队首是最近使用，队尾是最久未使用
```

---

#### 用在缓存模拟题中

如果有多个缓存组，可以写：

```cpp
vector<list<int>> cache(N);
vector<unordered_map<int, list<int>::iterator>> pos(N);
vector<unordered_set<int>> dirty(N);
```

含义：

```text
cache[g]：第 g 组的 LRU 链表
pos[g][a]：内存块 a 在第 g 组链表中的位置
dirty[g]：第 g 组里哪些块被写过，需要写回内存
```

判断命中：

```cpp
if (pos[g].count(a)) {
    // 命中
}
```

命中后移动到队首：

```cpp
cache[g].erase(pos[g][a]);
cache[g].push_front(a);
pos[g][a] = cache[g].begin();
```

未命中且需要替换队尾：

```cpp
int old = cache[g].back();
cache[g].pop_back();

pos[g].erase(old);
dirty[g].erase(old);
```

插入新块：

```cpp
cache[g].push_front(a);
pos[g][a] = cache[g].begin();
```

---

#### `unordered_map` 使用限制：没有排序，以及 key 必须可以哈希

`unordered_map` 的优点是查找快，均摊 $$O(1)$$。

但是它也有几个重要限制。

##### 1. 不排序

`unordered_map` 不会按照 key 从小到大排列。

```cpp
unordered_map<int, int> mp;
mp[3] = 30;
mp[1] = 10;
mp[2] = 20;

for (auto p : mp) {
    cout << p.first << " " << p.second << endl;
}
```

输出顺序是不确定的。

所以如果你需要：

- 按 key 从小到大遍历；
- 找最小 key；
- 找最大 key；
- 找前驱；
- 找后继；

那就不要用 `unordered_map`，应该用：

```cpp
map
set
```

##### 2. 不支持 `lower_bound` / `upper_bound`

`map` 支持：

```cpp
mp.lower_bound(x);
mp.upper_bound(x);
```

但 `unordered_map` 不支持。

因为哈希表内部没有顺序。

##### 3. key 必须可以哈希

`unordered_map<Key, Value>` 要求：

```text
Key 必须能被哈希，并且能判断相等
```

可以直接作为 key 的常见类型：

```cpp
unordered_map<int, int> mp1;
unordered_map<long long, int> mp2;
unordered_map<string, int> mp3;
```

但是下面这种很多环境下不能直接用：

```cpp
unordered_map<pair<int, int>, int> mp; // 可能 CE
```

因为标准库通常没有给 `pair<int, int>` 提供默认哈希。

考场建议把二维坐标压成 `long long`：

```cpp
long long encode(int x, int y) {
    return ((long long)x << 32) ^ (unsigned int)y;
}

unordered_map<long long, int> mp;
mp[encode(x, y)]++;
```

自定义结构体也不能直接作为 key：

```cpp
struct Point {
    int x, y;
};

unordered_map<Point, int> mp; // 通常 CE
```

如果非要用，需要自己写：

- `operator==`
- 哈希函数

示例：

```cpp
struct Point {
    int x, y;

    bool operator == (const Point& other) const {
        return x == other.x && y == other.y;
    }
};

struct PointHash {
    size_t operator()(const Point& p) const {
        return hash<long long>()(((long long)p.x << 32) ^ (unsigned int)p.y);
    }
};

unordered_map<Point, int, PointHash> mp;
```

---

#### `unordered_map + list` 考场口诀

```text
list 负责维护顺序；
unordered_map 负责 O(1) 定位；
删除 list 节点时，一定同步 erase 掉 unordered_map 里的记录；
编号小且连续，可以用 vector 存迭代器；
编号大且稀疏，必须用 unordered_map 存迭代器；
unordered_map 不排序，不能 lower_bound；
unordered_map 的 key 必须可以哈希。
```
