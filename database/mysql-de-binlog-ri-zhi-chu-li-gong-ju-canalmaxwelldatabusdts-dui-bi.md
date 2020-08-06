# MySQL 的 Binlog 日志处理工具（Canal/Maxwell/Databus/DTS）对比

[ref](https://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247492627&idx=1&sn=572ebb763ee0ec1991a1c46a05e036b8&chksm=fb080e87cc7f8791b2c2b0a00e722c33f4c4dd4ac3a74ffd3d9358d76d866a51582fa9e7b6b7&scene=0&xtrack=1#rd)

## **Canal**

定位：基于数据库增量日志解析，提供增量数据订阅&消费，目前主要支持了mysql。

原理：

* canal模拟mysql slave的交互协议，伪装自己为mysql slave，向mysql master发送dump协议
* mysql master收到dump请求，开始推送binary log给slave\(也就是canal\)
* canal解析binary log对象\(原始为byte流\)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/06/640-20200806114923545-114923.jpg)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/06/640-20200806114923650-114923.jpg)

整个parser过程大致可分为几步：

* Connection获取上一次解析成功的位置（如果第一次启动，则获取初始制定的位置或者是当前数据库的binlog位点）
* Connection建立连接，发生BINLOG\_DUMP命令
* Mysql开始推送Binary Log
* 接收到的Binary Log通过Binlog parser进行协议解析，补充一些特定信息
* 传递给EventSink模块进行数据存储，是一个阻塞操作，直到存储成功
* 存储成功后，定时记录Binary Log位置

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/06/640-20200806114923754-114923.jpg)

* 数据过滤：支持通配符的过滤模式，表名，字段内容等
* 数据路由/分发：解决1:n \(1个parser对应多个store的模式\)
* 数据归并：解决n:1 \(多个parser对应1个store\)
* 数据加工：在进入store之前进行额外的处理，比如join

## Maxwell

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/06/640-20200806114923863-114923.jpg)

canal 由Java开发，分为服务端和客户端，拥有众多的衍生应用，性能稳定，功能强大；canal 需要自己编写客户端来消费canal解析到的数据。

maxwell相对于canal的优势是使用简单，它直接将数据变更输出为json字符串，不需要再编写客户端。

## **Databus**

Databus是一种低延迟变化捕获系统，已成为LinkedIn数据处理管道不可或缺的一部分。Databus解决了可靠捕获，流动和处理主要数据更改的基本要求。Databus提供以下功能：

* 源与消费者之间的隔离
* 保证按顺序和至少一次交付具有高可用性
* 从更改流中的任意时间点开始消耗，包括整个数据的完全引导功能。
* 分区消费
* 源一致性保存

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/06/640-20200806114924060-114924.jpg)

## **阿里云的数据传输服务DTS**

数据传输服务（Data Transmission Service，简称DTS）是阿里云提供的一种支持 RDBMS（关系型数据库）、NoSQL、OLAP 等多种数据源之间数据交互的数据流服务。DTS提供了数据迁移、实时数据订阅及数据实时同步等多种数据传输能力，可实现不停服数据迁移、数据异地灾备、异地多活\(单元化\)、跨境数据同步、实时数据仓库、查询报表分流、缓存更新、异步消息通知等多种业务应用场景，助您构建高安全、可扩展、高可用的数据架构。

优势：数据传输（Data Transmission）服务 DTS 支持 RDBMS、NoSQL、OLAP 等多种数据源间的数据传输。它提供了数据迁移、实时数据订阅及数据实时同步等多种数据传输方式。相对于第三方数据流工具，数据传输服务 DTS 提供更丰富多样、高性能、高安全可靠的传输链路，同时它提供了诸多便利功能，极大得方便了传输链路的创建及管理。

个人理解：就是一个消息队列，会给你推送它包装过的sql对象，可以自己做个服务去解析这些sql对象。

免去部署维护的昂贵使用成本。DTS针对阿里云RDS（在线关系型数据库）、DRDS等产品进行了适配，解决了Binlog日志回收，主备切换、VPC网络切换等场景下的订阅高可用问题。同时，针对RDS进行了针对性的性能优化。出于稳定性、性能及成本的考虑，推荐使用。

