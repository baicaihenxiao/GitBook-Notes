# SpringBoot开发案例之整合Kafka实现消息队列

[https://blog.52itstyle.vip/archives/2868/](https://blog.52itstyle.vip/archives/2868/)

### 前言

最近在做一款秒杀的案例，涉及到了同步锁、数据库锁、分布式锁、进程内队列以及分布式消息队列，这里对SpringBoot集成Kafka实现消息队列做一个简单的记录。

### Kafka简介

Kafka是由Apache软件基金会开发的一个开源流处理平台，由Scala和Java编写。Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据。 这种动作（网页浏览，搜索和其他用户的行动）是在现代网络上的许多社会功能的一个关键因素。 这些数据通常是由于吞吐量的要求而通过处理日志和日志聚合来解决。 对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka的目的是通过Hadoop的并行加载机制来统一线上和离线的消息处理，也是为了通过集群来提供实时的消息。

Kafka是一种高吞吐量的分布式发布订阅消息系统，有如下特性：

* 通过O\(1\)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。
* 高吞吐量：即使是非常普通的硬件Kafka也可以支持每秒数百万的消息。
* 支持通过Kafka服务器和消费机集群来分区消息。
* 支持Hadoop并行数据加载。

[![kafka\_diagram.png](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/22/3263895972-133900.png)](https://blog.52itstyle.vip/usr/uploads/2018/05/3263895972.png)

[kafka\_diagram.png](https://blog.52itstyle.vip/usr/uploads/2018/05/3263895972.png)

### 术语介绍

* Broker

    Kafka集群包含一个或多个服务器，这种服务器被称为broker

* Topic

    每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）

* Partition

    Partition是物理上的概念，每个Topic包含一个或多个Partition.

* Producer

    负责发布消息到Kafka broker

* Consumer

    消息消费者，向Kafka broker读取消息的客户端。

* Consumer Group

    每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。

### Kafka安装

Kafka需要依赖JAVA环境运行，如何安装JDK这里不做介绍。

下载kafka：

```text
wget http://mirror.bit.edu.cn/apache/kafka/1.1.0/kafka_2.11-1.1.0.tgz
```

将包下载到执行目录并解压：

```text
cd /usr/local/
tar -xzvf kafka_2.11-0.10.0.1.tgz
```

修改kafka配置文件：

```text
cd kafka_2.11-0.10.0.1/config/
#编辑配置文件
vi server.properties
broker.id=0
#端口号、记得开启端口，云服务器要开放安全组
port=9092
#服务器IP地址，修改为自己的服务器IP
host.name=127.0.0.1
#zookeeper地址和端口， Kafka支持内置的Zookeeper和引用外部的Zookeeper
zookeeper.connect=localhost:2181
```

分别启动 kafka 和 zookeeper：

```text
./zookeeper-server-start.sh /usr/local/kafka_2.11-0.10.0.1/config/zookeeper.properties &
./kafka-server-start.sh /usr/local/kafka_2.11-0.10.0.1/config/server.properties &
```

### SpringBoot集成

pom.xml引入：

```markup
<!--kafka支持-->
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>1.3.5.RELEASE</version><!--$NO-MVN-MAN-VER$-->
</dependency>
```

application.properties配置：

```markup
#kafka相关配置
spring.kafka.bootstrap-servers=192.168.1.180:9092
#设置一个默认组
spring.kafka.consumer.group-id=0
#key-value序列化反序列化
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
#每次批量发送消息的数量
spring.kafka.producer.batch-size=65536
spring.kafka.producer.buffer-memory=524288
```

生产者KafkaSender：

```java
/**
 * 生产者
 * @author 科帮网 By https://blog.52itstyle.vip
 */
@Component
public class KafkaSender {
    @Autowired
    private KafkaTemplate<String,String> kafkaTemplate;

    /**
     * 发送消息到kafka
     */
    public void sendChannelMess(String channel, String message){
        kafkaTemplate.send(channel,message);
    }
}
```

消费者：

```java
/**
 * 消费者 spring-kafka 2.0 + 依赖JDK8
 * @author 科帮网 By https://blog.52itstyle.vip
 */
@Component
public class KafkaConsumer {
    /**
     * 监听seckill主题,有消息就读取
     * @param message
     */
    @KafkaListener(topics = {"seckill"})
    public void receiveMessage(String message){
        //收到通道的消息之后执行秒杀操作
    }
}
```

码云下载：[从0到1构建分布式秒杀系统](https://gitee.com/52itstyle/spring-boot-seckill)

### 参考

[http://kafka.apache.org/](http://kafka.apache.org/)

