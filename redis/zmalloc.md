## redis内存分配机制

* redis并不像memcache那样自行实现内存分配器，而是交由tcmalloc / jemalloc / malloc等分配器来管理;
* 如果使用的内存分配器没有`malloc_size`接口，则通过额外申请`PREFIX_SIZE`来存储内存块大小;
* 通过读取 /proc/{pid}/stat 来获取`RSS`;
* 通过自记录对齐内存大小叠加到原子变量来获取`used_memory`;
* 通过`RSS / used_memory`可以计算得到内存碎片比;
* 如果内存分配失败，会打印错误日志，然后`abort()`;
