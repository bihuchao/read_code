## redis伪随机数产生机制
采用pysam包中的drand48函数实现，具有设置随机数种子 / 获取下一个随机数的功能;

## 版本文件
通过`mkreleasehdr.sh`来生成`release.h`文件，包括以下参数：
* git sha: git commit id
* git dirty: git diff 函数
* build-id: 主机名等信息
通过这些信息拼装经过crc64可以得到一个build_id。
