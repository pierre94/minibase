# HBase-notes
> HBase的一些小笔记,用于辅助学习MiniBase

## ROWKey设计
### Rowkey长度原则（最好不超过16字节
Rowkey是一个二进制码流，Rowkey的长度被很多开发者建议说设计在10~100个字节，不过建议是越短越好，不要超过16个字节。
- 数据的持久化文件HFile中是按照KeyValue存储的,如果Rowkey过长比如100个字节，1000万列数据光Rowkey就要占用100*1000万=10亿个字节，将近1G数据，这会极大影响HFile的存储效率；
- MemStore将缓存部分数据到内存，如果Rowkey字段过长内存的有效利用率会降低，系统将无法缓存更多的数据，这会降低检索效率。因此Rowkey的字节长度越短越好。
- 目前操作系统是都是64位系统，内存8字节对齐。控制在16个字节，8字节的整数倍利用操作系统的最佳特性。

### Rowkey散列原则
> 把主键哈希后当成rowkey的头部
