# springboot + rabbitmq 用了消息确认机制，感觉掉坑里了

{% embed url="https://mp.weixin.qq.com/s/6LSaqgGqeC40Swxao1ik\_g" %}





最近部门号召大伙多组织一些技术分享会，说是要活跃公司的技术氛围，但早就看穿一切的我知道，这 T M 就是为了刷`KPI`。不过，话说回来这的确是件好事，与其开那些没味的扯皮会，多做技术交流还是很有助于个人成长的。

于是乎我主动报名参加了分享，咳咳咳~ ，真的不是为了那点`KPI`，就是想和大伙一起学习学习！

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/640-20200702201215422.png)

这次我分享的是 `springboot` + `rabbitmq` 如何实现消息确认机制，以及在实际开发中的一点踩坑经验，其实整体的内容比较简单，有时候事情就是这么神奇，越是简单的东西就越容易出错。

可以看到使用了 `RabbitMQ` 以后，我们的业务链路明显变长了，虽然做到了系统间的解耦，但可能造成消息丢失的场景也增加了。例如：

* 消息生产者 - &gt; rabbitmq服务器（消息发送失败）
* rabbitmq服务器自身故障导致消息丢失
* 消息消费者 - &gt; rabbitmq服务（消费消息失败）

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/640-20200702201215617.png)所以说能不使用中间件就尽量不要用，如果为了用而用只会徒增烦恼。开启消息确认机制以后，尽管很大程度上保证了消息的准确送达，但由于频繁的确认交互，`rabbitmq` 整体效率变低，吞吐量下降严重，不是非常重要的消息真心不建议你用消息确认机制。

下边我们先来实现`springboot` + `rabbitmq`消息确认机制，再对遇到的问题做具体分析。

### 一、准备环境

**1、引入 rabbitmq 依赖包**

```text
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**2、修改 application.properties 配置**

配置中需要开启 `发送端`和 `消费端` 的消息确认。

```text
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

# 发送者开启 confirm 确认机制
spring.rabbitmq.publisher-confirms=true
# 发送者开启 return 确认机制
spring.rabbitmq.publisher-returns=true
####################################################
# 设置消费端手动 ack
spring.rabbitmq.listener.simple.acknowledge-mode=manual
# 是否支持重试
spring.rabbitmq.listener.simple.retry.enabled=true
```

**3、定义 Exchange 和 Queue**

定义交换机 `confirmTestExchange` 和队列 `confirm_test_queue` ，并将队列绑定在交换机上。

```text
@Configuration
public class QueueConfig {

    @Bean(name = "confirmTestQueue")
    public Queue confirmTestQueue() {
        return new Queue("confirm_test_queue", true, false, false);
    }

    @Bean(name = "confirmTestExchange")
    public FanoutExchange confirmTestExchange() {
        return new FanoutExchange("confirmTestExchange");
    }

    @Bean
    public Binding confirmTestFanoutExchangeAndQueue(
            @Qualifier("confirmTestExchange") FanoutExchange confirmTestExchange,
            @Qualifier("confirmTestQueue") Queue confirmTestQueue) {
        return BindingBuilder.bind(confirmTestQueue).to(confirmTestExchange);
    }
}
```

> `rabbitmq` 的消息确认分为两部分：发送消息确认 和 消息接收确认。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/640-20200702201216129.png)在这里插入图片描述

### 二、消息发送确认

发送消息确认：用来确认生产者 `producer` 将消息发送到 `broker` ，`broker` 上的交换机 `exchange` 再投递给队列 `queue`的过程中，消息是否成功投递。

消息从 `producer` 到 `rabbitmq broker`有一个 `confirmCallback` 确认模式。

消息从 `exchange` 到 `queue` 投递失败有一个 `returnCallback` 退回模式。

我们可以利用这两个`Callback`来确保消的100%送达。

**1、 ConfirmCallback确认模式**

消息只要被 `rabbitmq broker` 接收到就会触发 `confirmCallback` 回调 。

```text
@Slf4j
@Component
public class ConfirmCallbackService implements RabbitTemplate.ConfirmCallback {

    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {

        if (!ack) {
            log.error("消息发送异常!");
        } else {
            log.info("发送者爸爸已经收到确认，correlationData={} ,ack={}, cause={}", correlationData.getId(), ack, cause);
        }
    }
}
```

实现接口 `ConfirmCallback` ，重写其`confirm()`方法，方法内有三个参数`correlationData`、`ack`、`cause`。

* `correlationData`：对象内部只有一个 `id` 属性，用来表示当前消息的唯一性。
* `ack`：消息投递到`broker` 的状态，`true`表示成功。
* `cause`：表示投递失败的原因。

但消息被 `broker` 接收到只能表示已经到达 MQ服务器，并不能保证消息一定会被投递到目标 `queue` 里。所以接下来需要用到 `returnCallback` 。

**2、 ReturnCallback 退回模式**

如果消息未能投递到目标 `queue` 里将触发回调 `returnCallback` ，一旦向 `queue` 投递消息未成功，这里一般会记录下当前消息的详细投递数据，方便后续做重发或者补偿等操作。

```text
@Slf4j
@Component
public class ReturnCallbackService implements RabbitTemplate.ReturnCallback {

    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        log.info("returnedMessage ===> replyCode={} ,replyText={} ,exchange={} ,routingKey={}", replyCode, replyText, exchange, routingKey);
    }
}
```

实现接口`ReturnCallback`，重写 `returnedMessage()` 方法，方法有五个参数`message`（消息体）、`replyCode`（响应code）、`replyText`（响应内容）、`exchange`（交换机）、`routingKey`（队列）。

下边是具体的消息发送，在`rabbitTemplate`中设置 `Confirm` 和 `Return` 回调，我们通过`setDeliveryMode()`对消息做持久化处理，为了后续测试创建一个 `CorrelationData`对象，添加一个`id` 为`10000000000`。

```text
@Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private ConfirmCallbackService confirmCallbackService;

    @Autowired
    private ReturnCallbackService returnCallbackService;

    public void sendMessage(String exchange, String routingKey, Object msg) {

        /**
         * 确保消息发送失败后可以重新返回到队列中
         * 注意：yml需要配置 publisher-returns: true
         */
        rabbitTemplate.setMandatory(true);

        /**
         * 消费者确认收到消息后，手动ack回执回调处理
         */
        rabbitTemplate.setConfirmCallback(confirmCallbackService);

        /**
         * 消息投递到队列失败回调处理
         */
        rabbitTemplate.setReturnCallback(returnCallbackService);

        /**
         * 发送消息
         */
        rabbitTemplate.convertAndSend(exchange, routingKey, msg,
                message -> {
                    message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
                    return message;
                },
                new CorrelationData(UUID.randomUUID().toString()));
    }
```

### 三、消息接收确认

消息接收确认要比消息发送确认简单一点，因为只有一个消息回执（`ack`）的过程。使用`@RabbitHandler`注解标注的方法要增加 `channel`\(信道\)、`message` 两个参数。

```text
@Slf4j
@Component
@RabbitListener(queues = "confirm_test_queue")
public class ReceiverMessage1 {

    @RabbitHandler
    public void processHandler(String msg, Channel channel, Message message) throws IOException {

        try {
            log.info("小富收到消息：{}", msg);

            //TODO 具体业务

            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);

        }  catch (Exception e) {

            if (message.getMessageProperties().getRedelivered()) {

                log.error("消息已重复处理失败,拒绝再次接收...");

                channel.basicReject(message.getMessageProperties().getDeliveryTag(), false); // 拒绝消息
            } else {

                log.error("消息即将再次返回队列处理...");

                channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true); 
            }
        }
    }
}
```

消费消息有三种回执方法，我们来分析一下每种方法的含义。

**1、basicAck**

`basicAck`：表示成功确认，使用此回执方法后，消息会被`rabbitmq broker` 删除。

```text
void basicAck(long deliveryTag, boolean multiple)
```

`deliveryTag`：表示消息投递序号，每次消费消息或者消息重新投递后，`deliveryTag`都会增加。手动消息确认模式下，我们可以对指定`deliveryTag`的消息进行`ack`、`nack`、`reject`等操作。

`multiple`：是否批量确认，值为 `true` 则会一次性 `ack`所有小于当前消息 `deliveryTag` 的消息。

**举个栗子：** 假设我先发送三条消息`deliveryTag`分别是5、6、7，可它们都没有被确认，当我发第四条消息此时`deliveryTag`为8，`multiple`设置为 true，会将5、6、7、8的消息全部进行确认。

**2、basicNack**

`basicNack` ：表示失败确认，一般在消费消息业务异常时用到此方法，可以将消息重新投递入队列。

```text
void basicNack(long deliveryTag, boolean multiple, boolean requeue)
```

`deliveryTag`：表示消息投递序号。

`multiple`：是否批量确认。

`requeue`：值为 `true` 消息将重新入队列。

**3、basicReject**

`basicReject`：拒绝消息，与`basicNack`区别在于不能进行批量操作，其他用法很相似。

```text
void basicReject(long deliveryTag, boolean requeue)
```

`deliveryTag`：表示消息投递序号。

`requeue`：值为 `true` 消息将重新入队列。

### 四、测试

发送消息测试一下消息确认机制是否生效，从执行结果上看发送者发消息后成功回调，消费端成功的消费了消息。![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/640-20200702201216348.png)用抓包工具`Wireshark` 观察一下`rabbitmq` amqp协议交互的变化，也多了 `ack` 的过程。![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/640-20200702201216920.png)

### 五、踩坑日志

**1、别忘确认消息**

这是一个非常没技术含量的坑，但却是非常容易犯错的地方。

开启消息确认机制，消费消息别忘了`channel.basicAck`，否则消息会一直存在，导致重复消费。![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/640-20200702201216979.png)

**2、消息无限投递**

在我最开始接触消息确认机制的时候，消费端代码就像下边这样写的，思路很简单：处理完业务逻辑后确认消息， `int a = 1 / 0` 发生异常后将消息重新投入队列。

```text
@RabbitHandler
    public void processHandler(String msg, Channel channel, Message message) throws IOException {

        try {
            log.info("消费者 2 号收到：{}", msg);

            int a = 1 / 0;

            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);

        } catch (Exception e) {

            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
        }
    }
```

但是有个问题是，业务代码一旦出现 `bug` 99.9%的情况是不会自动修复，一条消息会被无限投递进队列，消费端无限执行，导致了死循环。

在这里插入图片描述

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/02/640.gif)

本地的`CPU`被瞬间打满了，大家可以想象一下当时在生产环境导致服务死机，我是有多慌。

而且`rabbitmq management` 只有一条未被确认的消息。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/640-20200702201218688.png)

在这里插入图片描述

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/640-20200702201216979.png)

经过测试分析发现，当消息重新投递到消息队列时，这条消息不会回到队列尾部，仍是在队列头部。

消费者会立刻消费这条消息，业务处理再抛出异常，消息再重新入队，如此反复进行。导致消息队列处理出现阻塞，导致正常消息也无法运行。

而我们当时的解决方案是，先将消息进行应答，此时消息队列会删除该条消息，同时我们再次发送该消息到消息队列，异常消息就放在了消息队列尾部，这样既保证消息不会丢失，又保证了正常业务的进行。

```text
channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
// 重新发送消息到队尾
channel.basicPublish(message.getMessageProperties().getReceivedExchange(),
                    message.getMessageProperties().getReceivedRoutingKey(), MessageProperties.PERSISTENT_TEXT_PLAIN,
                    JSON.toJSONBytes(msg));
```

但这种方法并没有解决根本问题，错误消息还是会时不时报错，后面优化设置了消息重试次数，达到了重试上限以后，手动确认，队列删除此消息，并将消息持久化入`MySQL`并推送报警，进行人工处理和定时任务做补偿。

**3、重复消费**

如何保证 MQ 的消费是幂等性，这个需要根据具体业务而定，可以借助`MySQL`、或者`redis` 将消息持久化，通过再消息中的唯一性属性校验。

