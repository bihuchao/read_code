## redis双端链表
核心数据结构定义如下:
``` C++
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct listIter {
    listNode *next;
    int direction;
} listIter;

typedef struct list {
    listNode *head;
    listNode *tail; // 头尾指针都包含
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```

* 采用函数指针实现多态，包括(dup / free / match)，应用于不同的数据结构;
* 提供插入（包括任意位置/头/尾），删除，迭代器遍历，迭代器重置（头，尾），查找（o(n)）等功能;
