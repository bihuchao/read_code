## redis动态字符串 (dynamic string)

核心数据结构如下：

``` C++
typedef char *sds;

struct sdshdr {
    unsigned int len; // 实际使用长度
    unsigned int free; // 内存末尾free
    char buf[];
};
```

* 通过柔性数组（C++不支持）来实现;
* `typedef char* sds;`，向下兼容char*，可以使用cstdio中的库函数;
* 字符串扩缩容机制:
    初始化: 不预分配空间;
    扩容: `1 M`以下, 预留size空间 相当于`*2`; 1M以上，每次扩容`+1 M`
    缩容: 显式指定缩容;
