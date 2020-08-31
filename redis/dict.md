## redis dict实现

核心数据结构如下:
``` C++
// 字典item
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;

// hash table
typedef struct dictht {
    dictEntry **table;
    unsigned long size;     // 必须是pow(2, n)
    unsigned long sizemask; // 是pow(2,n) - 1
    unsigned long used;     // 已使用
} dictht;

// dict实现 双hash_table 用于rehash
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    // 为-1表示只使用ht[0], 否则表示处于rehash过程中
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    int iterators; /* number of iterators currently running */
} dict;
```

* hash table才用链地址法来解决hash冲突，str的hash函数为4字节的murmurhash函数;
* 其中size选取为2的n次方，这样mask的选取为size - 1，可以使用 & 来计算idx;
* 渐进式rehash：采用两个hash table来实现rehash, rehashid为-1表示未进行rehash，不为-1表示正在rehash;
* 通过逐步hash的方法来实现rehash, 避免eventloop卡住，所以需要拆分逻辑多次运行;
* rehash_n中包括对empty_bucket数目判断;
* 周期性函数中定时触发rehash 1 ms;
* 每次插入时判断是否需要扩容rehash, 扩容判断条件: 1) 装载因子>1 ; 2) 未在AOF/RDB过程中 或者 装载因子>5;
* 随机选出一个item: 需要考虑到rehash状态;
* 替换 / 查找 / 删除: 如果在rehash过程中，需要判断两个ht，否则1个;
* 增加: 如果在rehash过程中，只插入新ht，否则插入旧ht;
* shrink方法: 设置最合适大小, 上层ServerCron函数中会判断装载因子<0.1来触发缩容操作;

参考:
https://blog.csdn.net/belalds/article/details/93713491
