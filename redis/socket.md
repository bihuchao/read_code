## redis socket相关

### socket io

```
ssize_t syncWrite(int fd, char *ptr, ssize_t size, long long timeout);
// 往一个文件描述符中限制timeout写入 size buf
// 如果成功写入，返回size
// 如果timeout，返回 -1，标记 errno 为 ETIMEDOUT
// 如果遇到其他错误，返回 -1，维持默认 errno

ssize_t syncRead(int fd, char *ptr, ssize_t size, long long timeout);
// 返回size : 成功读入size
// 从一个文件描述符中timeout时间内导入size buf
// 返回0 : short read，可能是client关闭连接，read到EOF
// 返回-1 : 异常错误, 具体查看error
// 如果timeout，返回-1，标记错误为 ETIMEOUT


ssize_t syncReadLine(int fd, char *ptr, ssize_t size, long long timeout);
// 返回nread，表示读入一行\r\n，并删除\r\n 或者读满了buf
// 返回-1，错误见errno

```

* redis通过来循环操作同一个描述符实现读超时，写超时，这种方式会阻塞住其它连接;
* 感觉这里代码写的一般，尤其是syncReadLine 一次只读一个字节;

### socket api
* 阻塞/非阻塞设置
* 套接字选项设置
* tcp keepalive设置
``` bash
IPPROTO_TCP - TCP_KEEPIDLE / TCP_KEEPINTVL / TCP_KEEPCNT
SOL_SOCKET - SO_KEEPALIVE
```
* tcp nodelay设置
``` bash
IPPROTO_TCP - TCP_NODELAY
```
* 缓冲区设置
```
SOL_SOCKET SO_SNDBUF
```

* 发送超时设置
```
SOL_SOCKET SO_SNDTIMEO
```
* 设置地址复用
```
SOL_SOCKET SO_REUSEADDR
```

### background io
两个后台线程: 分别处理 fdatasync 和 close
参考
http://blog.chinaunix.net/uid-16569211-id-2996490.html
