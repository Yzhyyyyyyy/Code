# 📝 A.链表 (Linked List) 核心笔记

> **💡 核心思想**：把链表想象成一列**火车**。每个节点（Node）是一节**车厢**，节点里的 `next` 指针是连接车厢的**铁链**，而我们在外部定义的指针变量（如 `p`, `pos`, `pre`）是我们手里的**激光笔**。

## 🚂 一、 链表的本质：车厢与铁链
在 C 语言中，链表的节点长这样：
```c
typedef struct Node {
    char data;          // 📦 货箱：用来装真正的数据（比如字符 'A'）
    struct Node *next;  // 🔗 铁链：一个指针，永远指向下一节车厢的地址
} Node;
```

---

## ⚠️ 二、 核心易错点：`指针 = NULL` vs `指针->next = NULL`
这是链表操作中最容易报错（如段错误）的地方！必须彻底搞懂它们的本质区别：

### 1. `pos = NULL;` （关掉激光笔）
* **动作**：你把你手里的“激光笔”关掉了（或者指向了虚无）。
* **结果**：局部变量 `pos` 变成了 NULL，**但是火车本身没有任何变化！** 车厢之间依然用铁链死死地连着。
* **本质**：仅仅修改了栈内存上的局部变量，**毫无影响**，链表没断。

### 2. `pre->next = NULL;` （砍断铁链）
* **动作**：你顺着激光笔找到了 `pre` 指向的那节车厢，走过去，把这节车厢尾巴上连着下一节车厢的**铁链给砍断了**！
* **结果**：这节车厢的尾巴变成了空（NULL），**火车被真正截断了！** 后面的车厢彻底和前面失去了联系。
* **本质**：修改了堆内存中节点内部的数据，**成功截断**，链表变短了。

> **🎯 总结**：想要丢弃链表后面的垃圾数据，必须找到最后一个有用节点 `pre`，并执行 `pre->next = NULL;` 进行物理截断！

---

## 🛠️ 三、 从零手搓链表：尾插法建表
**口诀：找个列车长（tail），永远在最后面挂新车厢。**

建表核心“四步走”：
1. **开空间**：造一节新车厢，接在最后。
2. **移指针**：列车长 `tail` 走到新车厢上。
3. **存数据**：把货物放进新车厢。
4. **封口**：新车厢的铁链指向 NULL。

```c
Node* createList() {
    // 0. 准备工作：造一个空的火车头
    Node *head = (Node *)malloc(sizeof(Node)); 
    head->next = NULL; 
    Node *tail = head; // 列车长 tail 刚开始站在火车头上

    char c;
    while ((c = getchar()) != '\n') {
        // 【核心四步走】
        tail->next = (Node *)malloc(sizeof(Node)); // 1. 开空间并连上
        tail = tail->next;                         // 2. 移指针
        tail->data = c;                            // 3. 存数据
        tail->next = NULL;                         // 4. 封口（重要！）
    }
    return head; // 返回火车头
}
```

---

## 🚶‍♂️ 四、 巡视火车：链表的遍历
**口诀：找个替身巡检员，只要没走到虚无，就一直往下走。**

千万不要直接移动 `head`，否则会找不到火车的开头！要派一个替身指针 `p`：

```c
void printList(Node* head) {
    // 1. 找替身：巡检员 p 从火车头后面的第一节“真车厢”开始
    Node *p = head->next; 
    
    // 2. 只要 p 还没有走到虚无（NULL）
    while (p != NULL) {
        printf("%c ", p->data); // 3. 干活：看看当前车厢里装了啥
        
        p = p->next;            // 4. 往下走：顺着铁链，走到下一节车厢（灵魂代码✨）
    }
    printf("\n");
}
```

---

## 🏆 五、 进阶技巧：链表排序的最优解
在链表上直接做排序（比如交换节点）非常痛苦且容易死循环。
**最优解法：借用指针数组！**

**核心思路：**
1. **拆车厢**：遍历链表，把所有节点的**地址**存入一个数组 `Node **arr`。
2. **排数组**：对这个存满地址的数组使用 `qsort` 进行排序（按节点里的 `data` 比较）。
3. **连铁链**：按照排好序的数组顺序，把节点的 `next` 重新串起来。

```c
// C语言 qsort 比较函数（注意二级指针的解引用）
int cmp(const void *a, const void *b) {
    Node *nodeA = *(Node**)a; // 强转并解引用，拿到真正的 Node*
    Node *nodeB = *(Node**)b;
    return nodeA->data - nodeB->data; // 升序排列
}

void sortList(Node* head) {
    if (head->next == NULL || head->next->next == NULL) return;

    // 1. 统计长度
    int len = 0;
    Node *p = head->next;
    while (p != NULL) { len++; p = p->next; }

    // 2. 建指针数组并填入地址
    Node **arr = (Node **)malloc(len * sizeof(Node*));
    p = head->next;
    for (int i = 0; i < len; i++) {
        arr[i] = p;
        p = p->next;
    }

    // 3. 排序数组
    qsort(arr, len, sizeof(Node*), cmp);

    // 4. 重组链表（最爽的一步）
    Node *tail = head;
    for (int i = 0; i < len; i++) {
        tail->next = arr[i]; // 按排好的顺序挂车厢
        tail = tail->next;
    }
    tail->next = NULL; // 【致命细节】必须封口！防止死循环

    free(arr); // 释放辅助数组
}
```
