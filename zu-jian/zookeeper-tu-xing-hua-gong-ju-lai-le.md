# Zookeeper 图形化工具来了

[https://mp.weixin.qq.com/s?\_\_biz=MzkwNzI0MzQ2NQ==&mid=2247489450&idx=2&sn=00976ce0ba87c1d08d6b4963f90f1e29](https://mp.weixin.qq.com/s?__biz=MzkwNzI0MzQ2NQ==&mid=2247489450&idx=2&sn=00976ce0ba87c1d08d6b4963f90f1e29)



> ZooKeeper作为顶级分布式开源项目，应用非常广泛，Dubbo和Kafka这些知名的开源项目都在使用。之前只是听说过它，并没有仔细研究过。今天带大家来学习下ZooKeeper，主要从ZooKeeper的安装、可视化工具、应用三方面来介绍，希望对大家有所帮助！

## 简介

ZooKeeper是一款分布式协调框架，它可以为分布式系统提供一致性服务。ZooKeeper最初由Yahoo开发，后来捐献给了Apache基金会，现已成功Apache的顶级项目，目前在Github上有9.5k+Star。

## 分布式协调

要理解ZooKeeper我们首先需要了解下什么是`分布式协调`？这里拿Spring Cloud中注册中心的例子来说吧。

微服务（分布式）系统中有很多服务，相同的服务又有多个实例，我们在应用中可以通过服务名来负载均衡地调用服务，而这些服务有可能会挂掉，也有可能会有新的实例加入。此时我们就需要一个东西来做协调，保存好服务名称和可用实例调用IP的对应关系，此时注册中心就是一个分布式协调者的角色，而ZooKeeper就可以用来充当这个协调者。

## 安装

> ZooKeeper的安装无论是Windows还是Linux都是很方便的，我们先来学习下它的安装。

### Windows安装

* 首先下载ZooKeeper安装包，下载地址：[https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz](https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz)

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111638381-111638.png)

* 解压到指定目录，解压完成后目录结构如下；

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111645893-111646.png)

* 在`conf`目录下创建配置文件`zoo.cfg`，内容如下；

```text
# 设置心跳时间，单位毫秒
tickTime=2000
# 存储内存数据库快照的文件夹
dataDir=I:/developer/env/apache-zookeeper-3.7.0-bin/data
# 监听客户端连接的端口
clientPort=2181
```

* 进入`bin`目录，启动ZooKeeper服务；

```text
zkServer.cmd
```

* 服务启动成功后，控制台会输出如下信息。

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111652390-111652.png)

### Linux安装

* 使用Docker安装ZooKeeper无疑是最方便的，首先我们下载它的Docker镜像；

```text
docker pull zookeeper:3.7.0
```

* 创建好ZooKeeper的配置文件目录，并切换到该目录创建配置文件`zoo.cfg`；

```text
mkdir /mydata/zookeeper/conf/ -p
cd /mydata/zookeeper/conf/
touch zoo.cfg
```

* 配置文件`zoo.cfg`内容如下，直接使用VIM编辑即可；

```text
# 设置心跳时间，单位毫秒
tickTime=2000
# 存储内存数据库快照的文件夹
dataDir=/tmp/zookeeper
# 监听客户端连接的端口
clientPort=2181
```

* 运行ZooKeeper容器。

```text
docker run -p 2181:2181 --name zookeeper \
-v /mydata/zookeeper/conf/zoo.cfg:/conf/zoo.cfg \
-d zookeeper:3.7.0
```

## 命令行操作

> 接下来我们用命令行来操作下ZooKeeper，熟悉下ZooKeeper的使用。

* 首先使用`zkCli`命令行工具连接到ZooKeeper；

```text
zkCli.cmd -server 127.0.0.1:2181
```

* 通过`help`可以命令查看ZooKeeper的常用命令；

```text
[zk: 127.0.0.1:2181(CONNECTED) 0] help
ZooKeeper -server host:port -client-configuration properties-file cmd args
        addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE
        addauth scheme auth
        close
        config [-c] [-w] [-s]
        connect host:port
        create [-s] [-e] [-c] [-t ttl] path [data] [acl]
        delete [-v version] path
        deleteall path [-b batch size]
        delquota [-n|-b|-N|-B] path
        get [-s] [-w] path
        getAcl [-s] path
        getAllChildrenNumber path
        getEphemerals path
        history
        listquota path
        ls [-s] [-w] [-R] path
        printwatches on|off
        quit
        reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
        redo cmdno
        removewatches path [-c|-d|-a] [-l]
        set [-s] [-v version] path data
        setAcl [-s] [-v version] [-R] path acl
        setquota -n|-b|-N|-B val path
        stat [-w] path
        sync path
        version
        whoami
```

* 大家都知道Redis是通过`key-value`的形式存储数据的，而ZooKeeper是通过`znode-value`的形式存储数据的，znode有点像目录，而`/`目录就是ZooKeeper中的根目录，通过如下命令可以查看所有znode；

```text
[zk: 127.0.0.1:2181(CONNECTED) 1] ls /
[zookeeper]
```

* 创建一个znode叫做`/zk_test`，存储字符串`my_data`，这用起来有点像Redis；

```text
[zk: 127.0.0.1:2181(CONNECTED) 2] create /zk_test my_data
Created /zk_test
```

* 查看所有znode，可以看到`zk_test`这个znode；

```text
[zk: 127.0.0.1:2181(CONNECTED) 3] ls /
[zk_test, zookeeper]
```

* 获取znode中存储的数据；

```text
[zk: 127.0.0.1:2181(CONNECTED) 4] get /zk_test
my_data
```

* 修改znode中的数据；

```text
[zk: 127.0.0.1:2181(CONNECTED) 5] set /zk_test test_data
[zk: 127.0.0.1:2181(CONNECTED) 6] get /zk_test
test_data
```

* 删除znode中的数据；

```text
[zk: 127.0.0.1:2181(CONNECTED) 7] delete /zk_test
[zk: 127.0.0.1:2181(CONNECTED) 8] ls /
[zookeeper]
```

## 可视化管理

> `PrettyZoo`是一款基于 Apache Curator 和 JavaFX 实现的 Zookeeper 图形化管理客户端。颜值很高，推荐使用。

* 首先下载`PrettyZoo`的安装包，下载地址：[https://github.com/vran-dev/PrettyZoo/releases](https://github.com/vran-dev/PrettyZoo/releases)

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111138548-111138.png)

* 我们需要创建一个连接，连接到ZooKeeper，可以发现`PrettyZoo`是支持通过SSH通道连接的；

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111353707-111353.png)

* 双击连接，我们可以查看到ZooKeeper中存储的数据，很清楚的发现，ZooKeeper是按目录结构存储数据的；

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111348883-111349.png)

* 右键目录，我们可以创建和删除znode，有了这个工具，基本上可以和命令行操作说再见了；

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111344342-111344.png)

* 如果你还是觉得命令行比较炫酷的话，`PrettyZoo`也实现了命令行功能，打开命令行标签就可以愉快地敲命令了。

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111340173-111340.png)

## 节点类型

ZooKeeper中的节点（znode）是有生命周期的，这取决于节点的类型。类型有主要有下面四种：

* 持久节点（Persistent）：默认节点类型，节点创建后，会一直存在。
* 持久顺序节点（Persistent Sequential）：具有持久节点特性，节点名称后会增加自增数字后缀。
* 临时节点（Ephemeral）：临时存在，当创建节点的会话关闭时，节点被删除。
* 临时顺序节点（Ephemeral Sequential）：具有临时节点特性，节点名称后会增加自增数字后缀。

如果你用命令行创建节点的话，顺序特性对应`-s`选项，临时特性对应`-e`选项，比如如下命令：

```text
# 创建持久顺序节点
create -s /test/seq segText
# 创建临时节点
create -e /test/tmp tmpText
# 创建临时顺序节点
create -s -e /test/seqTmp setTmpText
```

创建成功后显示如下：

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111334436-111334.png)

如果你用`PrettyZoo`来创建的话，只要勾选一个选项即可。

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111329170-111329.png)

## 作为注册中心使用

> CAP是分布式架构中的重要理论，其包括一致性\(Consistency\)、可用性\(Availability\)和分区容忍性\(Partition tolerance\)。我们经常使用的Eureka支持AP，而ZooKeeper支持CP。接下来我们学习下ZooKeeper在Spring Cloud中作为注册中心的应用。

* ZooKeeper作为注册中心使用，用法基本和Eureka和Consul相同，首先我们需要在`pom.xml`中添加ZooKeeper的服务发现组件；

```text
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
</dependency>
```

* 之后修改配置文件`application.yml`，添加ZooKeeper相关配置；

```text
spring:
  cloud:
    zookeeper:
      # zookeeper连接地址
      connect-string: localhost:2181
      discovery:
        # 作为服务注册
        register: true
        # 注册时使用IP地址而不是hostname
        prefer-ip-address: true
```

* 这里还是使用《Spring Cloud学习教程》中的例子，有两个服务`zookeeper-ribbon-service`和`zookeeper-user-service`，前者通过Ribbon远程调用后者；

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111323194-111323.png)

* 分别启动两个服务，我们通过`PrettyZoo`可以发现，当ZooKeeper作为注册中心时，注册服务的名称、IP、端口都被存储到了里面；

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111317911-111318.png)

* 我们调用`zookeeper-ribbon-service`中的接口测试下，发现可以正常访问，接口地址：[http://localhost:8301/user/1](http://localhost:8301/user/1)

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111309350-111309.png)

* 如果这时候我们把`zookeeper-user-service`服务关掉的话，我们可以发现ZooKeeper会自动删除存储的数据；

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/24/640-20210624111301755-111301.png)

* 由此可以看出，ZooKeeper作为微服务的注册中心是通过临时节点来实现的，当服务上线时会向ZooKeeper中注册，当服务下线时会被ZooKeeper删除，保障了微服务的高可用。

## 总结

今天我们学习了下ZooKeeper的安装、可视化工具PrettyZoo的使用以及ZooKeeper在Spring Cloud中作为注册中心的应用。其实ZooKeeper在分布式系统中还有很多应用，比如说做分布式锁、实现选主功能、取代UUID来生成唯一ID，大家感兴趣的话可以深入研究下！

## 参考资料

官方文档：[https://zookeeper.apache.org/doc/current/zookeeperStarted.html](https://zookeeper.apache.org/doc/current/zookeeperStarted.html)



