# MiniBase学习笔记
> https://github.com/pierre94/minibase

- 原项目: https://github.com/openinx/minibase
- 资料: 《HBase原理与实践#设计存储引擎MiniBase》 https://weread.qq.com/web/reader/632326807192b335632d09ckc51323901dc51ce410c121b

## 接口
- put/get/delete (点写\查\删)
- scan(范围查询)

## 架构
MiniBase是一个标准的LSM树索引结构，分内存部分和磁盘部分。
![](img/README_images/arch.png)

### MemStore
- 客户端不断地写入数据，当MemStore的内存超过一定阈值时，MemStore会flush成一个磁盘文件。
- MemStore分成MutableMemstore和ImmutableMemstore两部分
    - MutableMemstore由一个ConcurrentSkipListMap组成，容许写入
    - ImmutableMemstore也是一个ConcurrentSkipListMap，但是不容许写入
- 这里设计两个小的MemStore，是为了防止在f lush的时候，MiniBase无法接收新的写入。假设只有一个MutableMemstore，那么一旦进入flush过程，MutableMemstore就无法写入，而此时新的写入就无法进行。
> 从源码不难看出,其实就是2个ConcurrentSkipListMap kvMap和snapshot,flush的时候将kvMap赋值给snapshot,然后启动一个新的ConcurrentSkipListMap

### DiskStore
- DiskStore，由多个DiskFile组成，每一个DiskFile就是一个磁盘文件。
- ImmutableMemstore执行完flush操作后，就会生成一个新的DiskFile，存放到DiskStore中.
- 为了有效控制DiskStore中的DiskFile个数，我们为MiniBase设计了Compaction策略。目前的Compaction策略非常简单——当DiskStore的DiskFile数量超过一个阈值时，就把所有的DiskFile进行Compact，最终合并成一个DiskFile。
 