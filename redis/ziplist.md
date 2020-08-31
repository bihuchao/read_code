## 编码双端链表

数据结构定义如下:
``` C++
// ziplist
#define ZIPLIST_BYTES(zl)       (*((uint32_t*)(zl)))
#define ZIPLIST_TAIL_OFFSET(zl) (*((uint32_t*)((zl)+sizeof(uint32_t))))
#define ZIPLIST_LENGTH(zl)      (*((uint16_t*)((zl)+sizeof(uint32_t)*2)))
// 总字节数 / 末尾元素偏移量 / 元素个数
#define ZIPLIST_HEADER_SIZE     (sizeof(uint32_t)*2+sizeof(uint16_t))

// ziplist entry
typedef struct zlentry {
    unsigned int prevrawlensize, prevrawlen;
    unsigned int lensize, len;
    unsigned int headersize;
    unsigned char encoding;
    unsigned char *p;
} zlentry;
```

* 编码数据列表，可以存储int / str;
* 每次操作都需要realloc内存;
* 总内存布局 : 总字节数(4) / 末尾元素offset(4) / 元素个数(2) / 元素s / 结束符(1);
* 元素内存布局 : 元素长度 / 元素encoding;
* 在插入元素时判断是否可以用int编码来节省内存;
