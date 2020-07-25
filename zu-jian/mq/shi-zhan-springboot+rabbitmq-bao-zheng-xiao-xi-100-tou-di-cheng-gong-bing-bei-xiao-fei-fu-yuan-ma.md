# 实战 \| SpringBoot+RabbitMQ ，保证消息100%投递成功并被消费（附源码）

[https://mp.weixin.qq.com/s?\_\_biz=MzAwNjQwNzU2NQ==&mid=2650344062&idx=1&sn=2666fc07c2449c78d1fb528ba4e107e9&chksm=83007e9cb477f78a9e1ac7e66774e930ffc22e06eec990ef678978819c44eebb345f87247e8a&mpshare=1&scene=1&srcid=0725oTnFrWTAjJDqjPGh9u51&sharer\_sharetime=1595688039354&sharer\_shareid=393f249533d421d13c2402bd43e74356\#rd](https://mp.weixin.qq.com/s?__biz=MzAwNjQwNzU2NQ==&mid=2650344062&idx=1&sn=2666fc07c2449c78d1fb528ba4e107e9&chksm=83007e9cb477f78a9e1ac7e66774e930ffc22e06eec990ef678978819c44eebb345f87247e8a&mpshare=1&scene=1&srcid=0725oTnFrWTAjJDqjPGh9u51&sharer_sharetime=1595688039354&sharer_shareid=393f249533d421d13c2402bd43e74356#rd)



> ### 来自：简书，作者：wangzaiplus
>
> 链接：[https://www.jianshu.com/p/dca01aad6bc8](https://www.jianshu.com/p/dca01aad6bc8)

## 一、先扔一张图

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-225742.jpeg)

**说明:**

本文涵盖了关于RabbitMQ很多方面的知识点, 如:

* 消息发送确认机制
* 消费确认机制
* 消息的重新投递
* 消费幂等性, 等等

这些都是围绕上面那张整体流程图展开的, 所以有必要先贴出来, 见图知意

## 二、实现思路

* 简略介绍163邮箱授权码的获取
* 编写发送邮件工具类
* 编写RabbitMQ配置文件
* 生产者发起调用
* 消费者发送邮件
* 定时任务定时拉取投递失败的消息, 重新投递
* 各种异常情况的测试验证

拓展: 使用动态代理实现消费端幂等性验证和消息确认\(ack\)

## 三、项目介绍

* springboot版本2.1.5.RELEASE, 旧版本可能有些配置属性不能使用, 需要以代码形式进行配置
* RabbitMQ版本3.7.15
* MailUtil: 发送邮件工具类
* RabbitConfig: rabbitmq相关配置
* TestServiceImpl: 生产者, 发送消息
* MailConsumer: 消费者, 消费消息, 发送邮件
* ResendMsg: 定时任务, 重新投递发送失败的消息

说明: 上面是核心代码, MsgLogService mapper xml等均未贴出, 完整代码可以参考GitHub上的源码，地址在文末。

## 四、代码实现

**1、163邮箱授权码的获取, 如图:**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225742146-225742.jpeg)

该授权码就是配置文件spring.mail.password需要的密码

**2、pom**

```text
        <!--mq-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <!--mail-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```

**3、rabbitmq、邮箱配置**

```text
# rabbitmq
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
# 开启confirms回调 P -> Exchange
spring.rabbitmq.publisher-confirms=true
# 开启returnedMessage回调 Exchange -> Queue
spring.rabbitmq.publisher-returns=true
# 设置手动确认(ack) Queue -> C
spring.rabbitmq.listener.simple.acknowledge-mode=manual
spring.rabbitmq.listener.simple.prefetch=100

# mail
spring.mail.host=smtp.163.com
spring.mail.username=18621142249@163.com
spring.mail.password=123456wangzai
spring.mail.from=18621142249@163.com
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
```

说明: password即授权码, username和from要一致

**4、表结构**

```text
CREATE TABLE `msg_log` (
  `msg_id` varchar(255) NOT NULL DEFAULT '' COMMENT '消息唯一标识',
  `msg` text COMMENT '消息体, json格式化',
  `exchange` varchar(255) NOT NULL DEFAULT '' COMMENT '交换机',
  `routing_key` varchar(255) NOT NULL DEFAULT '' COMMENT '路由键',
  `status` int(11) NOT NULL DEFAULT '0' COMMENT '状态: 0投递中 1投递成功 2投递失败 3已消费',
  `try_count` int(11) NOT NULL DEFAULT '0' COMMENT '重试次数',
  `next_try_time` datetime DEFAULT NULL COMMENT '下一次重试时间',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`msg_id`),
  UNIQUE KEY `unq_msg_id` (`msg_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='消息投递日志';
```

说明: exchange routing\_key字段是在定时任务重新投递消息时需要用到的

**5、MailUtil**

```text
@Component
@Slf4j
public class MailUtil {

    @Value("${spring.mail.from}")
    private String from;

    @Autowired
    private JavaMailSender mailSender;

    /**
     * 发送简单邮件
     *
     * @param mail
     */
    public boolean send(Mail mail) {
        String to = mail.getTo();// 目标邮箱
        String title = mail.getTitle();// 邮件标题
        String content = mail.getContent();// 邮件正文

        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(title);
        message.setText(content);

        try {
            mailSender.send(message);
            log.info("邮件发送成功");
            return true;
        } catch (MailException e) {
            log.error("邮件发送失败, to: {}, title: {}", to, title, e);
            return false;
        }
    }

}
```

**6、RabbitConfig**

```text
@Configuration
@Slf4j
public class RabbitConfig {

    @Autowired
    private CachingConnectionFactory connectionFactory;

    @Autowired
    private MsgLogService msgLogService;

    @Bean
    public RabbitTemplate rabbitTemplate() {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(converter());

        // 消息是否成功发送到Exchange
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (ack) {
                log.info("消息成功发送到Exchange");
                String msgId = correlationData.getId();
                msgLogService.updateStatus(msgId, Constant.MsgLogStatus.DELIVER_SUCCESS);
            } else {
                log.info("消息发送到Exchange失败, {}, cause: {}", correlationData, cause);
            }
        });

        // 触发setReturnCallback回调必须设置mandatory=true, 否则Exchange没有找到Queue就会丢弃掉消息, 而不会触发回调
        rabbitTemplate.setMandatory(true);
        // 消息是否从Exchange路由到Queue, 注意: 这是一个失败回调, 只有消息从Exchange路由到Queue失败才会回调这个方法
        rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            log.info("消息从Exchange路由到Queue失败: exchange: {}, route: {}, replyCode: {}, replyText: {}, message: {}", exchange, routingKey, replyCode, replyText, message);
        });

        return rabbitTemplate;
    }

    @Bean
    public Jackson2JsonMessageConverter converter() {
        return new Jackson2JsonMessageConverter();
    }

    // 发送邮件
    public static final String MAIL_QUEUE_NAME = "mail.queue";
    public static final String MAIL_EXCHANGE_NAME = "mail.exchange";
    public static final String MAIL_ROUTING_KEY_NAME = "mail.routing.key";

    @Bean
    public Queue mailQueue() {
        return new Queue(MAIL_QUEUE_NAME, true);
    }

    @Bean
    public DirectExchange mailExchange() {
        return new DirectExchange(MAIL_EXCHANGE_NAME, true, false);
    }

    @Bean
    public Binding mailBinding() {
        return BindingBuilder.bind(mailQueue()).to(mailExchange()).with(MAIL_ROUTING_KEY_NAME);
    }

}
```

**7、TestServiceImpl生产消息**

```text
@Service
public class TestServiceImpl implements TestService {

    @Autowired
    private MsgLogMapper msgLogMapper;

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Override
    public ServerResponse send(Mail mail) {
        String msgId = RandomUtil.UUID32();
        mail.setMsgId(msgId);

        MsgLog msgLog = new MsgLog(msgId, mail, RabbitConfig.MAIL_EXCHANGE_NAME, RabbitConfig.MAIL_ROUTING_KEY_NAME);
        msgLogMapper.insert(msgLog);// 消息入库

        CorrelationData correlationData = new CorrelationData(msgId);
        rabbitTemplate.convertAndSend(RabbitConfig.MAIL_EXCHANGE_NAME, RabbitConfig.MAIL_ROUTING_KEY_NAME, MessageHelper.objToMsg(mail), correlationData);// 发送消息

        return ServerResponse.success(ResponseCode.MAIL_SEND_SUCCESS.getMsg());
    }

}
```

**8、MailConsumer消费消息, 发送邮件**

```text
@Component
@Slf4j
public class MailConsumer {

    @Autowired
    private MsgLogService msgLogService;

    @Autowired
    private MailUtil mailUtil;

    @RabbitListener(queues = RabbitConfig.MAIL_QUEUE_NAME)
    public void consume(Message message, Channel channel) throws IOException {
        Mail mail = MessageHelper.msgToObj(message, Mail.class);
        log.info("收到消息: {}", mail.toString());

        String msgId = mail.getMsgId();

        MsgLog msgLog = msgLogService.selectByMsgId(msgId);
        if (null == msgLog || msgLog.getStatus().equals(Constant.MsgLogStatus.CONSUMED_SUCCESS)) {// 消费幂等性
            log.info("重复消费, msgId: {}", msgId);
            return;
        }

        MessageProperties properties = message.getMessageProperties();
        long tag = properties.getDeliveryTag();

        boolean success = mailUtil.send(mail);
        if (success) {
            msgLogService.updateStatus(msgId, Constant.MsgLogStatus.CONSUMED_SUCCESS);
            channel.basicAck(tag, false);// 消费确认
        } else {
            channel.basicNack(tag, false, true);
        }
    }

}
```

说明: 其实就完成了3件事: 1.保证消费幂等性, 2.发送邮件, 3.更新消息状态, 手动ack

**9、ResendMsg定时任务重新投递发送失败的消息**

```text
@Component
@Slf4j
public class ResendMsg {

    @Autowired
    private MsgLogService msgLogService;

    @Autowired
    private RabbitTemplate rabbitTemplate;

    // 最大投递次数
    private static final int MAX_TRY_COUNT = 3;

    /**
     * 每30s拉取投递失败的消息, 重新投递
     */
    @Scheduled(cron = "0/30 * * * * ?")
    public void resend() {
        log.info("开始执行定时任务(重新投递消息)");

        List<MsgLog> msgLogs = msgLogService.selectTimeoutMsg();
        msgLogs.forEach(msgLog -> {
            String msgId = msgLog.getMsgId();
            if (msgLog.getTryCount() >= MAX_TRY_COUNT) {
                msgLogService.updateStatus(msgId, Constant.MsgLogStatus.DELIVER_FAIL);
                log.info("超过最大重试次数, 消息投递失败, msgId: {}", msgId);
            } else {
                msgLogService.updateTryCount(msgId, msgLog.getNextTryTime());// 投递次数+1

                CorrelationData correlationData = new CorrelationData(msgId);
                rabbitTemplate.convertAndSend(msgLog.getExchange(), msgLog.getRoutingKey(), MessageHelper.objToMsg(msgLog.getMsg()), correlationData);// 重新投递

                log.info("第 " + (msgLog.getTryCount() + 1) + " 次重新投递消息");
            }
        });

        log.info("定时任务执行结束(重新投递消息)");
    }

}
```

说明: 每一条消息都和exchange routingKey绑定, 所有消息重投共用这一个定时任务即可

## 五、基本测试

OK, 目前为止, 代码准备就绪, 现在进行正常流程的测试

**1、发送请求:**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225742499-225742.jpeg)

**2、后台日志:**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225743068-225743.jpeg)

**3、数据库消息记录:**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225743355-225743.jpeg)

状态为3, 表明已消费, 消息重试次数为0, 表明一次投递就成功了

**4、查看邮箱**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225743701-225743.jpeg)

发送成功

## 六、各种异常情况测试

步骤一罗列了很多关于RabbitMQ的知识点, 很重要, 很核心, 而本文也涉及到了这些知识点的实现, 接下来就通过异常测试进行验证\(这些验证都是围绕本文开头扔的那张流程图展开的, 很重要, 所以, 再贴一遍\)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-225742.jpeg)

**1、验证消息发送到Exchange失败情况下的回调, 对应上图P -&gt; X**

如何验证? 可以随便指定一个不存在的交换机名称, 请求接口, 看是否会触发回调

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225744264-225744.jpeg)

发送失败, 原因: reply-code=404, reply-text=NOT\_FOUND - no exchange 'mail.exchangeabcd' in vhost '/', 该回调能够保证消息正确发送到Exchange, 测试完成

**2、验证消息从Exchange路由到Queue失败情况下的回调, 对应上图X -&gt; Q**

同理, 修改一下路由键为不存在的即可, 路由失败, 触发回调

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225744705-225744.jpeg)

发送失败, 原因: route: mail.routing.keyabcd, replyCode: 312, replyText: NO\_ROUTE

**3、验证在手动ack模式下, 消费端必须进行手动确认\(ack\), 否则消息会一直保存在队列中, 直到被消费, 对应上图Q -&gt; C**

将消费端代码channel.basicAck\(tag, false\);// 消费确认注释掉, 查看控制台和rabbitmq管控台

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225745097-225745.jpeg)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225745446-225745.jpeg)

可以看到, 虽然消息确实被消费了, 但是由于是手动确认模式, 而最后又没手动确认, 所以, 消息仍被rabbitmq保存, 所以, 手动ack能够保证消息一定被消费, 但一定要记得basicAck

**4、验证消费端幂等性**

接着上一步, 去掉注释, 重启服务器, 由于有一条未被ack的消息, 所以重启后监听到消息, 进行消费, 但是由于消费前会判断该消息的状态是否未被消费, 发现status=3, 即已消费, 所以, 直接return, 这样就保证了消费端的幂等性, 即使由于网络等原因投递成功而未触发回调, 从而多次投递, 也不会重复消费进而发生业务异常

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225745857-225745.jpeg)

**5、验证消费端发生异常消息也不会丢失**

很显然, 消费端代码可能发生异常, 如果不做处理, 业务没正确执行, 消息却不见了, 给我们感觉就是消息丢失了, 由于我们消费端代码做了异常捕获, 业务异常时, 会触发: channel.basicNack\(tag, false, true\);, 这样会告诉rabbitmq该消息消费失败, 需要重新入队, 可以重新投递到其他正常的消费端进行消费, 从而保证消息不被丢失

测试: send方法直接返回false即可\(这里跟抛出异常一个意思\)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225746323-225746.jpeg)

可以看到, 由于channel.basicNack\(tag, false, true\), 未被ack的消息\(unacked\)会重新入队并被消费, 这样就保证了消息不会走丢

**6、验证定时任务的消息重投**

实际应用场景中, 可能由于网络原因, 或者消息未被持久化MQ就宕机了, 使得投递确认的回调方法ConfirmCallback没有被执行, 从而导致数据库该消息状态一直是投递中的状态, 此时就需要进行消息重投, 即使也许消息已经被消费了

定时任务只是保证消息100%投递成功, 而多次投递的消费幂等性需要消费端自己保证

我们可以将回调和消费成功后更新消息状态的代码注释掉, 开启定时任务, 查看是否重投

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225746743-225746.jpeg)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/25/640-20200725225747199-225747.png)

可以看到, 消息会重投3次, 超过3次放弃, 将消息状态置为投递失败状态, 出现这种非正常情况, 就需要人工介入排查原因

## 七、拓展: 使用动态代理实现消费端幂等性验证和消费确认\(ack\)

不知道大家发现没有, 在MailConsumer中, 真正的业务逻辑其实只是发送邮件mailUtil.send\(mail\)而已, 但我们又不得不在调用send方法之前校验消费幂等性, 发送后, 还要更新消息状态为"已消费"状态, 并手动ack, 实际项目中, 可能还有很多生产者-消费者的应用场景, 如记录日志, 发送短信等等, 都需要rabbitmq, 如果每次都写这些重复的公用代码, 没必要, 也难以维护, 所以, 我们可以将公共代码抽离出来, 让核心业务逻辑只关心自己的实现, 而不用做其他操作, 其实就是AOP

为达到这个目的, 有很多方法, 可以用spring aop, 可以用拦截器, 可以用静态代理, 也可以用动态代理, 在这里, 我用的是动态代理

目录结构如下:

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/25/640-20200725225747543-225747.jpeg)

核心代码就是代理的实现, 这里就不把所有代码贴出来了, 只是提供一个思路, 我们要尽可能地把代码写的更简洁更优雅

## 八、总结

发送邮件其实很简单, 但深究起来其实有很多需要注意和完善的点, 一个看似很小的知识点, 也可以引申出很多问题, 甚至涉及到方方面面, 这些都需要自己踩坑, 当然我这代码肯定还有很多不完善和需要优化的点, 希望小伙伴多多提意见和建议。

我的代码都是经过自测验证过的, 图也都是一点一点自己画的或认真截的, 希望小伙伴能学到一点东西, 路过的点个赞或点个关注呗, 谢谢。

项目Github, 欢迎`fork` [https://github.com/wangzaiplus/springboot/tree/wxw](https://github.com/wangzaiplus/springboot/tree/wxw)

```text
·END·
```

