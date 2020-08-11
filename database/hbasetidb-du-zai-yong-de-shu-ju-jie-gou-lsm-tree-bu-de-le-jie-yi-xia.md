# HBase/TiDB都在用的数据结构：LSM Tree，不得了解一下？

[https://mp.weixin.qq.com/s?\_\_biz=MzU3NzU0NjYzMA==&mid=2247484240&idx=1&sn=6af9c7c662b8afcad04bfb09a531a66a&chksm=fd03b3b9ca743aaf2ab4ce3a66909b32eaec24dc752728e9fbf4dfe2aa0c2f7352a32d6d58e1&mpshare=1&scene=1&srcid=0810YWQxZIYhBC9SqKN0YzXJ&sharer\_sharetime=1597113204611&sharer\_shareid=393f249533d421d13c2402bd43e74356\#rd](https://mp.weixin.qq.com/s?__biz=MzU3NzU0NjYzMA==&mid=2247484240&idx=1&sn=6af9c7c662b8afcad04bfb09a531a66a&chksm=fd03b3b9ca743aaf2ab4ce3a66909b32eaec24dc752728e9fbf4dfe2aa0c2f7352a32d6d58e1&mpshare=1&scene=1&srcid=0810YWQxZIYhBC9SqKN0YzXJ&sharer_sharetime=1597113204611&sharer_shareid=393f249533d421d13c2402bd43e74356#rd)

LSM Tree（Log-structured merge-tree）广泛应用在HBase，TiDB等诸多数据库和存储引擎上，我们先来看一下它的一些应用：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-215505.png)

参考资料【4】

这么牛X的名单，你不想了解下LSM Tree吗？装X之前，我们先来了解一些基本概念。

设计数据存储系统可能需要考虑的一些问题有：ACID，RUM（Read,Write,Memory）。

## ACID

**ACID** 相信小伙伴都被面试官问过，我想简单讨论的一点是：如何 持久化数据 才能保证数据写入的 事务性 和 读写性能？

事务性可简单理解为：1.数据必须持久化。2.一次数据的写入返回给用户 写入成功就一定成功，失败就一定失败。

读写性能可简单理解为：一次读 或 一次写 需要的IO次数，因为访问速率：CPU&gt;&gt;内存&gt;&gt;SSD/磁盘。

对于单机存储，最简单的方式当然是：写一条就持久化一条，读一条就遍历一遍所有数据，然后返回。当然没人这么干，在内存中我们都还知道用个HashMap呢。

拿Mysql InnoDB举例子：

读性能体现在数据的索引在磁盘上主要用B+树来保证。

写性能体现在运用 **WAL机制** 来避免每次写都去更新B+树上的全量索引和数据内容，而是通过redo log记录下来每次写的增量内容，顺序将redo log写入磁盘。同时在内存中记录下来本次写入应该在B+树上更新的脏页数据，然后在一定条件下触发脏页的刷盘。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215504891-215505.png)

redo log是数据增删改的实时增量数据，顺序写入保证了写性能和事务性。磁盘上的B+树是数据增删改的全量数据，通过定时脏页刷盘保证这份数据的完整性。

这里的问题是：虽然通过WAL机制，提高了写入性能，但是仍然避免不了脏页数据定时刷盘的随机IO写入。

因为数据在磁盘上是以block为单位存储的（比如4K，16K）,假如你更新了Order表一条数据，又更新了Product表一条数据，脏页刷盘时，这两条数据在磁盘上的block位置可能差了十万八千里，就变成了随机IO的写入。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215505041-215505.png)参考资料【3】

## RUM

RUM（Read,Write,Memory）类似分布式领域的CAP定理。如果想得到三者中的两者性能，必须以牺牲第三者的性能为代价。衡量这三者的性能指标有：读放大、写放大、空间放大。

**读放大**：是指每次查询读取磁盘的次数。如果每次查询需要查询5个page \(或block\)，则读放大是5.

**写放大**：是指数据写入存储设备的量和数据写入数据库的量之比。如果用户写入数据库的大小是10MB，而实际写入磁盘的大小是30MB，则写放大是3.另外SSD的写入次数是有限的，写放大会减少SSD的寿命。

**空间放大**：是指数据在存储设备的总量和数据在数据库的总量之比。如果用户往数据库上存10MB，而数据库实际在磁盘上用了100MB去存，则空间放大就是10.

通常，数据结构可以最多优化这其中的两者，如B+树有更少的读放大，LSM Tree有更少的写放大。

下面我们将介绍LSM Tree的结构，它是如何解决 B+树中“全量数据的随机写入” 问题的；在RUM问题上，又该如何优化LSM Tree。

## 为什么要有LSM Tree

LSM Tree（Log-structured merge-tree）起源于1996年的一篇论文：The log-structured merge-tree \(LSM-tree\)。

当时的背景是：为一张数据增长很快的历史数据表设计一种存储结构，使得它能够解决：在内存不足，磁盘随机IO太慢下的严重写入性能问题。

如果我们先后修改两条数据，那么在脏数据块落盘时，不是一条条随机写入，而是以一个脏块批量落盘时，就能解决 B+树中“全量数据的随机写入” 问题了。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215505167-215505.png)

所以大佬设计了这么一种结构：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215505284-215505.png)

Ck tree是一个有序的树状结构，数据的写入流转从C0 tree 内存开始，不断被合并到磁盘上的更大容量的Ck tree上。这种方式写入是顺序写的，增删改操作都是append。这样一次I/O可以进行多条数据的写入，从而充分利用每一次写I/O。

merge所做的事情有：当上一棵树容量不足时，1.将上棵树中要删除的数据删除掉，进行GC回收。2.合并数据的最新版本，使得Ck tree不会过度膨胀。

这种初始结构存在的问题有：随着数据量的增大，merge费劲，没有好的索引结构，读放大严重。现代的LSM Tree结构如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215505399-215505.png)

**MemTable**：是LSM Tree在内存中还未刷盘的写入数据，这里面的数据可以修改。是一个有序的树状结构，如RocksDB中的跳表。

**SSTable（Sorted String Table）**：是一个键是有序的，存储字符串形式键值对的磁盘文件。当SSTable太大时，可以将其划分为多个block；我们需要通过 下图中的Index 记录下来每个block的起始位置，那么就可以构建每个SSTable的稀疏索引。这样在读SSTable前，通过Index结构就知道要读取的数据块磁盘位置了。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215505515-215505.png)

### LSM Tree读数据

读数据 包括点读和范围读，我们分开讨论。假设LSM Tree中的key为\[0-99\]，

**点读**：是指准确地取出一条数据，如get\(23\),则其示意图为：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215505676-215505.png)

按照图中标示的顺序，会先读取内存，在从Level0依次往高层读取，直到找到key=23的数据。

**范围读**：是指读取一个范围内的数据，如读取\[23-40\]内的所有数据（ select \* from t where key &gt;= 23 && key &lt;=40\),

则需要做的是在每一层SSTable找到这个范围内的第一个block,然后遍历block块的数据，直到不符合范围条件。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215505832-215505.png)参考资料【4】

**“\*\***ReadOnly MemTable\*\*：如RocksDB，在MemTable 和 Level0数据之间还有一个内存的ReadOnly MemTable结构。它是LSM Tree在内存中还未刷盘的写入数据，这里面的数据不可以修改。是当MemTable到达一定量需要刷到SSTable时，由MemTable完整copy下来的。这样可避免从MemTable直接刷到SSTable时的并发竞争问题。

### LSM Tree写数据

在LSM Tree写入数据时，我们把ReadOnly MemTable画上，示意图如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215505943-215506.png)

### 

当有写请求时，数据会先写入MemTable,同时写入 WAL Log；当MemTable空间不足时，触发ReadOnly MemTable copy,同时写入WAL Log；然后刷新到磁盘Level0的SSTable中。当低层SSTable空间不足时，不断通过Compaction和高层SSTable进行Merge。

### LSM Tree合并策略

LSM Tree有两种合并策略，Tiering 和 Leveling。我们看图说话：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215506051-215506.png)

**Tiering**的特点是：每层可能有N个重叠的sorted runs，当上一层的sorted runs 要merge下来时，不会和当前层重叠的sorted runs马上合并，而是等到当前层空间不足时，才会合并，然后再merge到下一层。

Tiering这种策略可以减小写放大，但是以读放大和空间放大为代价。

**Leveling**的特点是：每层只有一个sorted run，当上一层的sorted run 要merge下来时，会马上和当前层的sorted run合并。

Leveling这种策略可以减小读放大和空间放大，但以写放大为代价。

## LSM Tree的优化

参考资料\[2-4\]中的大佬们提到了不少优化方式，我这里从读，合并方面简要总结下LSM Tree的优化。

### 点读的优化

尽管我们可以通过SSTable的内存block稀疏索引结构简单判断key是否可能存在于block中，但如上get\(23\)，如果level1读不到，仍需要往下读level2,level3。（因为数据是按照增删改的时间顺序一层层往下落盘的，如果一个key不存在低level中，可能存在于更早的高level中）。这样的点读IO次数较多，读放大严重。

**布隆过滤器**

对于精确查询，hash的时间复杂度为 O\(1\)，那么可以通过布隆过滤器来优化。我们只需要查询时先过一遍布隆过滤器，就能排除掉一定不存在的key了，避免不必要的IO查询。

**“**布隆过滤器：是通过hash算法来判断一个key是否存在于某个集合中，布隆过滤器通常是一个bit数组，用针对一个key多次hash算法确定的多个bit值来表示key是否存在。多个bit值全为1，则表示key可能存在，否则不存在。好处是多个bit值通常比直接存储一个存在的key占用空间小，省内存。

**分层布隆过滤器**

上述布隆过滤器是假设每层数据都使用相同的布隆过滤器来进行过滤，而数据随着层数的增加通常是指数级增长的，如果使低层的数据使用更精确的布隆过滤器（所需bit数更多，但是精确度更高）,高层的数据使用稍微不那么精确的布隆过滤器，则整体点查的效率将提高。

参考资料【3】中Monkey做了上述优化，即使随着数据量和层数的增大，点查仍能保持在一个稳定的时间内。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215506224-215506.png)

### 

### 范围读的优化

Hash的不足就是无法支持范围查询索引，优化方式有：

**并行查询**

因为读数据不存在竞争问题，可以并行读每层的数据，缩短整体查询时间，但是这种方式实际并没有缩短IO次数。

**前缀布隆过滤器**

前缀布隆过滤器是以key的前缀来Hash构建布隆过滤器，而不是key本身的值。这样可以起到过滤 like Monica\* 这样的查询条件作用。RocksDB有做此优化。

### 合并的优化

上面提到了两种合并策略Tiering 和 Leveling。Tiering可以减小写放大，Leveling可以减小读放大和空间放大。参考资料【3】中提到了数据库所采用的合并策略走势如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215506438-215506.png)

随着数据量的增大，整条曲线会上移。优化的重点在于：如何结合两种合并策略，使曲线整体下移。

**Lazy Leveling**

参考资料【3】中Dostoevsky采用了一种Lazy Leveling的策略：它是对低层数据采用Tiering合并策略，对高层数据采用Leveling合并策略。从而权衡了读写性能。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215506550-215506.png)

### 

### RUM的取舍

RUM的取舍和权衡类似于分布式领域的CAP，在特定的场景采用适合的优化手段才能使整体性能最优。参考资料【3】中的大佬还提到了Fluid Merge策略。233酱表示虽过了眼瘾，但仍是个CRUD渣渣。

关于RUM的优化，233酱最后放一张图，希望对你我有所启发。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/11/640-20200811215506666-215506.png)

## 后记

LSM Tree是这两年一直听到的名词，但是一直没有花时间了解它。233酱趁这次机会现学现卖，对其有了更深一步的了解。

对本文内容感兴趣的小伙伴可进一步阅读参考资料，如果文中有错误或者不理解的地方也欢迎小伙伴们和我交流指正。

另外，如果觉得有收获也请帮忙 **四连【关注，点赞，在看，转发】** 支持一下233酱，谢谢～

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/11/640-215506.jpg)

参考资料：

\[1\].[https://kernelmaker.github.io/lsm-tree](https://kernelmaker.github.io/lsm-tree)

\[2\].[https://www.youtube.com/watch?v=aKAJMd0iKtI](https://www.youtube.com/watch?v=aKAJMd0iKtI)

\[3\].[https://www.youtube.com/watch?v=b6SI8VbcT4w](https://www.youtube.com/watch?v=b6SI8VbcT4w)

\[4\].[https://www.bilibili.com/video/BV1fb411x7zz?from=search&seid=8679886514280300283](https://www.bilibili.com/video/BV1fb411x7zz?from=search&seid=8679886514280300283)

\[5\].[https://tikv.github.io/deep-dive-tikv/key-value-engine/B-Tree-vs-Log-Structured-Merge-Tree.html](https://tikv.github.io/deep-dive-tikv/key-value-engine/B-Tree-vs-Log-Structured-Merge-Tree.html)

