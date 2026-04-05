# C++ string解析与大模拟实战指南

本文档总结了在处理类似 CCF CSP 201709-3 "JSON 查询"、201403-3 "命令行选项" 或 "网页模板生成" 这样的大模拟题时，关于 C++ `std::string` 的核心知识点、常见陷阱以及最优的数据结构设计思路。

---

## 一、 `std::string` 的传参、返回与核心工具

### 1. 作为函数的参数和返回值
*   **作为输入参数（最佳实践）**：使用**常引用 `const string&`**。这既保证了不会发生耗时的深拷贝，又保证了函数内部不会意外修改原字符串。
    ```cpp
    void parse_key(const string& str) { /* 只能读取，速度最快 */ }
    ```
*   **作为返回值**：直接返回 `string` 即可。现代 C++ 编译器有 RVO（返回值优化）机制，通常不会发生多余的拷贝。
    ```cpp
    string get_value() { return "result"; }
    ```

### 2. 必备的 `string` 工具函数与实战套路

*   **查找 `find()` 与 `string::npos`（极其重要）**：
    *   `find("子串")` 返回的是匹配到的子串的**第一个字符的下标（索引，从 0 开始）**。
    *   **致命陷阱**：如果找遍整个字符串都没找到，它**不会**返回 `0`，也**不是**简单的 `-1`，而是返回一个特殊的常量：**`string::npos`**。
*   **截取 `substr()`**：
    *   `substr(pos, len)`：从 `pos` 开始截取 `len` 个字符。
    *   `substr(pos)`：**神级用法**，省略第二个参数（默认值为 `string::npos`），表示从 `pos` 开始一直截取到字符串末尾。
*   **替换 `replace()`**：
    *   `replace(pos, len, new_str)`：从下标 `pos` 开始，将长度为 `len` 的一段内容，**原地替换**为 `new_str`。
*   **删除 `erase()`**：
    *   `erase(pos, len)`：从下标 `pos` 开始，删除 `len` 个字符。
    *   `erase(pos)`：从下标 `pos` 开始，**一直删除到字符串末尾**。

#### 🏆 实战套路：基于 `find` 的模板循环替换（大模拟必考）
在处理带有占位符的字符串（如 `Hello {{ name }}!`）时，标准的**查找、提取、替换**三步走循环模板如下：

```cpp
string s = "Hello {{ name }}, welcome to {{ place }}!";

// 只要还能找到 "{{ "，就一直循环处理
while (s.find("{{ ") != string::npos) {
    int start_pos = s.find("{{ "); // 找到 "{{ " 的首字符下标
    int end_pos = s.find(" }}");   // 找到对应 " }}" 的首字符下标
    
    // 1. 配合 substr 抠出中间的变量名
    // 变量名的起始位置是 start_pos + 3 (跳过 "{{ ")
    // 变量名的长度是 end_pos - (start_pos + 3)
    string var_name = s.substr(start_pos + 3, end_pos - start_pos - 3);
    
    // 2. 去 Map 里查真实的变量值 (假设 real_val 是查到的值)
    string real_val = myMap[var_name]; 
    
    // 3. 配合 replace 将整个占位符替换为真实值
    // 要替换的总长度是 end_pos + 3 - start_pos (包含前后的括号和空格)
    s.replace(start_pos, end_pos + 3 - start_pos, real_val);
}
```

---

## 二、 数据结构设计：降维打击的“路径展平”

**问题**：如何存储 JSON 结构？用 `map` 存键值对，再用邻接表存层级关系可以吗？

**回答**：对于通用的 JSON 解析库，建树是正确的。但对于**查询全是绝对路径**的算法题，建树过于复杂。最优解是**“路径展平（Flattening）”**。

### 核心思路：只用一个 `std::map<string, string>`
不要建树，直接把**从根节点到叶子节点的完整绝对路径**作为 Key。

**示例 JSON**：
```json
{
  "address": {
    "city": "New York",
    "street": "21 2nd Street"
  }
}
```
**展平后的 Map 存储**：
```cpp
map["address"] = "{OBJECT}";          // 遇到 '{' 时存入，用于处理 OBJECT 查询
map["address.city"] = "New York";     // 绝对路径作为 Key
map["address.street"] = "21 2nd Street";
```

### 优势：完美处理多分支
即使 `address` 下有多个分支，因为我们存的是完整的绝对路径（`address.city`），每一条路径都是唯一的，所以不会产生冲突。查询时，只需 `map.count(query_str)` 一步到位。

---

## 三、 字符串解析的致命陷阱：转义字符

**问题：控制台输入的 `\"` 会被编译器自动当做完整的转义字符处理吗？可以直接用 `find("\"")` 找字符串结尾吗？**

**回答**：**绝对不行！这是极其致命的错觉！**

### 1. 编译时 vs 运行时
*   在 C++ 代码里写 `string s = "a\"b";`，编译器在**编译时**会处理转义。
*   通过 `cin` 或 `getline` 从控制台读取输入的 `\"` 时，C++ 会将其视为**两个独立的字符**：反斜杠 `\` 和双引号 `"`。

### 2. 为什么 `find` 会“瞎眼”？
如果你用 `find("\"")` 寻找结尾：
*   遇到 `"name": "a\"b"`：`find` 会找到中间那个被转义的 `"`，导致字符串被提前截断。
*   **连续反斜杠陷阱**：遇到 `"a\\"`（实际内容是 `a\`）。程序会误以为最后的 `"` 被转义了，但实际上前面的 `\` 是用来转义另一个 `\` 的，最后的 `"` 就是真正的结尾！

### 3. 正确的破局方法：逐字符扫描（状态机）
处理带有转义字符的字符串，**必须放弃 `find`，使用从左到右的逐字符扫描**。只要遇到 `\`，就无脑跳过它，并把下一个字符当做普通内容吃掉。

**标准解析模板**：
```cpp
string extract_string(const string& json_str, int& i) {
    string result = "";
    i++; // 跳过开头的双引号
    
    while (i < json_str.length()) {
        if (json_str[i] == '\\') {
            // 遇到反斜杠，跳过它，直接吃掉下一个被转义的字符
            i++; 
            result.push_back(json_str[i]); 
        } 
        else if (json_str[i] == '"') {
            // 遇到未被转义的双引号，字符串真正结束
            break; 
        } 
        else {
            result.push_back(json_str[i]);
        }
        i++;
    }
    return result;
}
```

---

## 四、 字符串读取与切分神技：`getline` 避坑与 `stringstream`

### 1. 案发现场：为什么 `getline` 会读到空字符串？
当使用 `getline(cin, line)` 读取包含空格的整行命令时，如果前面用过 `cin >> n`，第一条命令往往会读到一个空字符串。
**原因解析**：`cin >> n` 读取数字时极其“懒惰”，它只拿走数字，**把用户敲下的回车符 `\n` 留在了输入缓冲区里**。紧接着执行 `getline` 时，它一探头发现缓冲区第一个字符就是 `\n`，误以为这一行已经结束，直接返回 `""`。

### 2. 终极防坑指南：如何完美读取字符串？

#### 🌟 推荐方案 1：使用 `cin >> ws`（竞技编程最常用、最优雅）
`ws` (Whitespace) 会吃掉输入管道里所有开头的空白字符（包括空格、回车 `\r`、换行 `\n`、制表符 `\t`），直到遇到真正的字符为止。
```cpp
int n;
cin >> n;
for (int i = 0; i < n; i++) {
    string line;
    getline(cin >> ws, line); // 完美跳过前面的所有换行和空格！
}
```

#### 🌟 推荐方案 2：使用 `cin.ignore()`（最标准、最严谨）
如果题目要求保留行首的空格（例如读取带缩进的代码），就不能用 `ws`，必须手动吃掉上一行的换行符。
```cpp
#include <limits>
int n;
cin >> n;          
cin.ignore(numeric_limits<streamsize>::max(), '\n'); // 清空缓冲区直到遇到 '\n'

for (int i = 0; i < n; i++) {
    string line;
    getline(cin, line); 
}
```

#### ⚠️ 必须注意的附加补丁：斩杀 Windows 幽灵回车 `\r`
**背景**：洛谷等评测机是 Linux 系统（只认 `\n` 作为换行），但出题人往往在 Windows 下造数据（换行是 `\r\n`）。当 Linux 的 `getline` 读到 `\r\n` 时，它会在 `\n` 处停下，**把 `\r` 留在了字符串的末尾**！
**对策**：只要用了 `getline`，立刻在下一行补上这句“全平台免疫”代码：
```cpp
getline(cin >> ws, line);
if (!line.empty() && line.back() == '\r') line.pop_back(); // 斩杀 \r
```

### 3. `stringstream`：优雅的“切词机”
拿到一整行字符串后，如果需要按空格分割单词，手动写 `for` 循环极其繁琐。此时，`std::stringstream` 是降维打击级别的工具。
*   **头文件**：`#include <sstream>`
*   **标准解析模板**：
    ```cpp
    void parse_command(const string& line) {
        stringstream ss(line); // 将长字符串塞入切词机
        string token;
        
        // 核心魔法：只要还能切出单词（自动过滤多余空格），循环就继续
        while (ss >> token) {
            if (token == "-w") {
                string param;
                if (ss >> param) { // 顺手再切一个单词作为参数
                    cout << "参数是: " << param << endl;
                }
            }
        }
    }
    ```

---

## 五、 `std::map` 的底层逻辑与遍历神技

### 1. 天生自带字典序排序
`std::map` 的底层是一个**红黑树**。当你把数据塞进 `map<string, string>` 时，它会**自动按照 Key（键）从小到大排好序**（字典序）。
*   **实战优势**：题目要求“按字母顺序输出”时，**完全不需要写排序代码**，直接遍历输出即可！

### 2. 优雅遍历模板（C++11 及以上）
利用 `auto` 和范围 `for` 循环，直接遍历 `map` 中的 `pair`：
```cpp
map<string, string> ans;
ans["-w"] = "10";
ans["-a"] = "";

for (auto& item : ans) {
    cout << item.first << " " << item.second << endl;
}
```

---

## 六、 字符串的隐蔽细节：`""` 与 `" "` 的天壤之别

在表示“没有值”或“空状态”时，千万不要混淆空字符串和空格字符串！

*   **`""`（空字符串）**：完全空的盒子，没有任何字符，`length()` 为 **0**。
*   **`" "`（空格字符串）**：装了一个空格字符（ASCII 32）的盒子，`length()` 为 **1**。

### 实战避坑：为什么无参选项必须存 `""`？
在命令行解析题中，对于无参数的选项（如 `-a`），我们需要记录它出现过。
*   **错误做法**：存空格 `ans["-a"] = " ";`。输出时会变成 `-a  `（末尾多出一个空格），导致 CSP 评测机报**格式错误（0分）**。
*   **满分做法**：存空字符串 `ans["-a"] = "";`。这完美表达了“存在但无附加值”。

--- 
**🚀 终极读入黄金法则总结**：
1. **字符串中间没有空格**（如 `a=1+2;`）：**无脑用 `cin >> s;`**，最安全省心。
2. **字符串中间有空格，且不关心行首空格**：用 **`getline(cin >> ws, s);`**。
3. **字符串中间有空格，且必须保留行首空格**：用 `cin.ignore()` 清理上一行后，再用 **`getline(cin, s);`**。
4. **只要用了 `getline`**：必须加上 **`if (!s.empty() && s.back() == '\r') s.pop_back();`** 防御 Windows 幽灵回车！
5. **拿到整行数据后**：用 **`stringstream`** 进行无痛切分。
