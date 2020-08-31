## 编码哈希表zipmap
数据结构
``` C++
char*;
```

* 内存布局: 总长度(1) / (keylen / key / valuelen / freelen / value)s;
* 总长度 <= 254 有用; > 254 需要遍历整个数据结构得到length;
* keylen / valuelen如果 <253 (1字节) >253(4字节);
