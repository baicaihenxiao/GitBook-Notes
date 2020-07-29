# 面试官这夺命连环12问

[https://mp.weixin.qq.com/s?\_\_biz=MzUzMTA2NTU2Ng==&mid=2247496183&idx=1&sn=5140d409222d91812f4316ef32897abe&chksm=fa4a8e46cd3d0750863318cfcaf6dfe777d87380ab2f6ab2b240c59f0a91079cca7f67431709&scene=0&xtrack=1\#rd](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247496183&idx=1&sn=5140d409222d91812f4316ef32897abe&chksm=fa4a8e46cd3d0750863318cfcaf6dfe777d87380ab2f6ab2b240c59f0a91079cca7f67431709&scene=0&xtrack=1#rd)

面试官: 同学，我看你每个项目中都用到了Redis,你能说说你是怎样使用Redis的吗？  


小A同学: 主要用来做缓存，分布式Session, 阅读量/点赞数统计

面试官: 嗯，好的，Redis如何做持久化的？ 

小A同学: bgsave做全量持久化到RDB二进制文件中，aof做增量持久化，存储的是文本协议数据。  

面试官：它们的优缺点呢？

小A同学：rdb二进制文件启动加载速度可以更快，aof要重放命令，所以速度比较慢

![](https://mmbiz.qpic.cn/mmbiz_jpg/LJXhQIbo682DgqVEqRiaf9AYkGrwGq9PEVvmZ2icMGGeC1eUUuLRwWnhHDyGuM1P5w6LBapOpaZznuWv2Ns0mqRg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

面试官: Redis持久化期间，主进程还能对外提供服务吗？

小A同学: 能

面试官：那Redis如何处理新写入的数据呢，这个数据也会直接进行持久化吗？ 

小A同学：。。。这个可能吧!     

面试官: Reids可以设置最大内存大小，如果数据达到了内存最大限制，Redis如何处理呢？

小A同学：可以配置淘汰策略 LRU 或者 LFU 淘汰策略。

面试官：Redis 的LRU算法实现原理，可以讲讲吗？

小A同学：这个不太清楚。

![](https://mmbiz.qpic.cn/mmbiz_jpg/LJXhQIbo682DgqVEqRiaf9AYkGrwGq9PETFicFcCIKicgLicK8r8zuTxQKS38qb4UkY17Eq5ShwicrGoJEbuDxlcsZw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

面试官: Redis 核心数据类型有哪些？

小A同学: string, hash, list, set, zset.

面试官：存储数据用 string 类型 和 hash 类型，你是如何选择的呢？

小A同学：string 对大量字段的对象中的某个数据进行获取，需要进行整体的数据获取，在客户端完成反序列化，而hash可以获取指定字段获取数据。所以根据访问需求来选择。

面试官：还有其他的考虑吗？ 

小A同学：没有

面试官: zset 底层的实现原理有了解过吗？

小A同学: 好像是跳表实现的吧！

面试官: 你能讲讲它的实现原理以及时间复杂度分析吗？

小A同学：这个不太清楚。 

![](https://mmbiz.qpic.cn/mmbiz_jpg/LJXhQIbo682DgqVEqRiaf9AYkGrwGq9PEzb7BPrrlnNUa3pANSs9ZWKzhxOwJViaQFVyngd3wxMl1Ot6au0Gt2ZQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

面试官: 你能说说缓存穿透是怎么回事吗？

小A同学：要查询的数据，缓存中不存在，直接打到了数据库，这种请求如果很多的话，全都穿透到数据库, 就会导致数据库奔溃，

面试官：解决方案呢？

小A同学：可以用布隆过滤器来阻挡。

面试官：布隆过滤器的实现原理是什么？能讲讲么？

小A同学：这个不太清楚。

面试官：好的，感谢你参加我们公司的面试，咱们今天就先到这里

![](https://mmbiz.qpic.cn/mmbiz_jpg/LJXhQIbo682DgqVEqRiaf9AYkGrwGq9PE0nRQRibRy5CCPHlqXZJnFvUjj9aJlLNc8HSFJVpibsdykfDvhAly0zSA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

