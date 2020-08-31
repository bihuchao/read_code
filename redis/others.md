## redis伪随机数产生机制
采用pysam包中的drand48函数实现，具有设置随机数种子 / 获取下一个随机数的功能;

## 版本文件
通过`mkreleasehdr.sh`来生成`release.h`文件，包括以下参数：
* git sha: git commit id
* git dirty: git diff 函数
* build-id: 主机名等信息
通过这些信息拼装经过crc64可以得到一个build_id。

## hash算法
* redis包含sha1 / crc32 / crc64 / murmurhash2
* sha1: 一串字符串 => 40位字符串 40 B
* crc : 循环冗余校验, 一块内存 => 16 b / 64 b
* mrumurhash2 : 一块内存 => 4 B

## 大小端转换
* Redis使用小端，如果是大端的话，需要进行转换

## 内存测试
* 包括内存寻址A测试和随机读写R测试
