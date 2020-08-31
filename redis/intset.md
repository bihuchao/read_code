## 有序整数集合

数据结构定义如下:
``` C++
typedef struct intset {
    uint32_t encoding; // 单个元素占用字节数 2 / 4 / 8
    uint32_t length; // 长度
    int8_t contents[]; // 元素实际存储
} intset;
```

* 用有序数组实现的整数集合, 可以根据元素内容动态扩容 16 b => 32 b => 64 b;
* 每次插入/删除都会触发内存realloc, 采用二分查找;
* 可以节省内存;
