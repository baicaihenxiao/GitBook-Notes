# Java开发常用技术栈盘点

{% embed url="https://mp.weixin.qq.com/s?__biz=MzUzMzQ2MDIyMA==&mid=2247485292&idx=2&sn=4564732355fdb36ba0d2ec5348d7bdc4&chksm=faa2e2ffcdd56be97777a5a3fb3d89e133aac564619b3c3bbd318e8ee80dff8e770637435f8e&scene=158#rd" %}



### 1. 前言

最近很多人私下询问我常用的**Java**开发技术栈，所以今天就总结一波平常使用的**Java**技术栈。

### 2. JDK

**JDK** 的版本现在大部分转移到了**8**，超前一点的使用**11**，请认准**LTS**版本！不要生产上使用**9**、 **10**、**12**、**13**、**14**。在**Oracle JDK**和**Open JDK**之间推荐使用**Open JDK**，避免引起不必要的商业纠纷。Amazon Corretto 、Alibaba Dragonwell 都不错。7 以上不用太考虑兼容问题，不过最好测试一波再迁移。

### 2. Web 框架

主流还是**Servlet**系列的**Spring MVC**为主。**Structs**应该只有老项目在用。响应式框架**Spring Webflux**开始进入视野，尝试的人、问的人逐渐多了起来。建议有志于抓住未来方向的同学了解一下。

### 3. Web 容器

目前应该还是**Tomcat**最多，但是近几年红帽的**Undertow**也起来了，**Jetty**实际生产并没有优势，测试可能会用。有能力的公司会选择**Netty**自行实现高性能的 Web 容器。

### 4. ORM 框架

现在**Mybatis**在国内依然是老大的地位，国外却很少有相关的教程。其次是**JPA**体系，主要包括**Spring Data JPA** 、**Hibernate**。有兴趣的话可以去看一下**JOOQ**。随着响应式编程的兴起，**JDBC**开始出现了潜在的对手**R2DBC**，需要持续关注动向。

### 5. Spring

谈到**Java**离不开**Spring**，**Spring**生态的统治地位依然不可动摇。目前单体应用还是**Spring Boot**一把梭，微服务**Spring Cloud**体系还是占绝对优势。但是你的项目真适合搞微服务吗？**Spring**近年来开始转向响应式，无论**Webflux**,还是**R2DBC**,以及更近的**RSocket**都是**Spring**官方力推的一些响应式框架或协议。所以响应式必须列入你的知识清单了。

### 6. 数据库

大部分还是**Mysql**、但是**MSSQL**、**PostgreSQL**也用的不少。国产云原生数据库**TiDB**的发展也不可小视。作为文档数据库**Mongo**虽然过去两年爆出了一些安全问题，但是依然领导着这个领域。内存型数据库**Redis**依然在缓存领域占据重要的地位，**Memcached**、**Hazelcast** 也经常出现在视野中。

### 7. 搜索引擎

在搜索引擎领域**Lucene**及其两个衍生品**Solr**和**ElasticSearch**占据绝对优势，**ElasticSearch**更加活跃一些。

### 8.后端模板引擎

在前后端分离已经流行的今天，模板引擎的生存空间再一次被压缩，目前我最多用它们来搞搞代码生成器。已经很少在使用它们了，**Freemarker**、**Velocity** **Thymeleaf**越来越少被提及了。

### 9. 工作流

常用的名气大的主要是**Activity**和**Flowable**。

### 9. 其它语言无关的中间件

消息队列主要是**Kafka**、**RocketMQ**、**RabbitMQ**,老牌**ActiveMQ**开始没落，**Yahoo**捐献给**Apache**的**Pulsar**不知道为什么没有像**zookeeper**一样买账的。**Nginx**依然是高性能**Web**服务器、代理服务器的首选。

这就是我对**Java**当前常用技术栈的一些看法和观点。如果你有不同的意见和补充请留言讨论，也欢迎转发让更多人看到。
