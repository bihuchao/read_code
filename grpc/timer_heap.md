## grpc时间堆设计
grpc中使用时间小顶堆进行定时事件处理。时间堆关键数据结构如下：

```
typedef struct {
  int heap_index; // 记录时间堆中位置，便于删除
  // ... 其他字段省略
} grpc_timer;

typedef struct {
  grpc_timer** timers;
  uint32_t timer_count;
  uint32_t timer_capacity;
} grpc_timer_heap;
```

扩容: 当`timer_count == timer_capacity`时，进行扩容，扩容为`max(timer_count + 1, timer_count * 1.5)`;
缩容: 当`timer_count >= 8`并且装载因子`timer_count/timer_capacity < 0.25`时，进行缩容，缩容为`2 * timer_count`;
add: 添加到末尾，然后向上调整`adjust_upwards`;
del: 腾挪最后元素到空置位置，判断向上调整`adjust_upwards`和`adjust_downwards`并执行;
pop: 删除首元素，向下调整`adjust_downwards`;
