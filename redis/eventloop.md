## redis eventloop调度机制

相关数据结构如下:
``` C++

// 文件事件
typedef struct aeFileEvent {
    int mask; // one of AE_(READABLE|WRITABLE) 只支持 读 / 写，不支持ET
    aeFileProc *rfileProc; // 读写回调函数
    aeFileProc *wfileProc;
    void *clientData; // 回调函数参数
} aeFileEvent;

// 时间事件
typedef struct aeTimeEvent {
    long long id; /* time event identifier. */
    long when_sec; /* seconds */
    long when_ms; /* milliseconds */
    aeTimeProc *timeProc;
    aeEventFinalizerProc *finalizerProc;
    void *clientData;
    struct aeTimeEvent *next;
} aeTimeEvent;

// 文件触发事件
typedef struct aeFiredEvent {
    int fd;
    int mask; // 触发何种事件
} aeFiredEvent;

// EventLoop核心数据结构
typedef struct aeEventLoop {
    // 一些多路复用API需要，例如select poll
    int maxfd;   /* highest file descriptor currently registered */
    // 预先分配链表大小
    int setsize; /* max number of file descriptors tracked */ // size
    long long timeEventNextId;
    time_t lastTime;     /* Used to detect system clock skew */

    aeFileEvent *events;        // redis文件事件链表
    aeFiredEvent *fired;        // 触发文件事件链表
    aeTimeEvent *timeEventHead; // 时间事件无序链表，用于server定时事件

    int stop;
    void *apidata; // 多路复用API相关数据 (例如epoll / select / kqueue)

    aeBeforeSleepProc *beforesleep; // 每次进行wait前调用的函数
} aeEventLoop;
```

* 采用`include .c文件来指定不同的多路复用API`, redis中包括了kqueue/epoll_wait等多路复用API;
* 文件事件链表和触发文件事件链表都是预先分配的，大小为 setsize;
* 通过控制多路复用api的超时时间来处理各时间事件:
    如果存在时间事件，则选取最近时间事件的时间间隔作为超时等待时间；
    否则则阻塞或者不阻塞来调用多路复用API;
* 当从多路复用API中唤起后，首先会调用个触发文件事件的回调读写函数，其次处理时间事件函数
* redis采用无序的时间事件链表来保存时间事件，搜索 / pop / 删除 / 添加效率不高
    是因为这个时间事件链表只用来存放服务定时事件，本身要求也不高
* 时间事件的处理是采用遍历无序时间事件链表，根据时间是否达到依次调用，如果为周期性时间还需要更新时间戳
