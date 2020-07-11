# 如何实现一个可复用的分布式事务消息架构方案？

[https://mp.weixin.qq.com/s/seqXV7cACdA3ZH208Ef\_tg](https://mp.weixin.qq.com/s/seqXV7cACdA3ZH208Ef_tg)

## **前提**

分布式事务是微服务实践中一个比较棘手的问题，在笔者所实施的微服务实践方案中，都采用了折中或者规避强一致性的方案。参考`Ebay`多年前提出的本地消息表方案，基于`RabbitMQ`和`MySQL（JDBC）`做了轻量级的封装，实现了低入侵性的事务消息模块。本文的内容就是详细分析整个方案的设计思路和实施。环境依赖如下：

* JDK1.8+
* spring-boot-start-web:2.x.x
* spring-boot-start-jdbc:2.x.x
* spring-boot-start-amqp:2.x.x
* HikariCP:3.x.x（spring-boot-start-jdbc自带）
* mysql-connector-java:5.1.48
* redisson:3.12.1

## **方案设计思路**

事务消息原则上只适合弱一致性（或者说`最终一致性`）的场景，常见的弱一致性场景如：

* 用户服务完成了注册动作，向短信服务推送一条营销相关的消息。
* 信贷体系中，订单服务保存订单完毕，向审批服务推送一条待审批的订单记录信息。
* ……

**强一致性的场景一般不应该选用事务消息。**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/11/640-20200711182849029-182849.png)

一般情况下，要求强一致性说明要严格同步，也就是所有操作必须同时成功或者同时失败，这样就会引入同步带来的额外消耗。

如果一个事务消息模块设计合理，补偿、查询、监控等等功能都完毕，由于系统交互是异步的，整体吞吐要比严格同步高。在笔者负责的业务系统中基于事务消息使用还定制了一条基本原则：**消息内容正确的前提下，消费方出现异常需要自理**。

> 简单来说就是：上游保证了自身的业务正确性，成功推送了正确的消息到RabbitMQ就认为上游义务已经结束。

为了降低代码的入侵性，事务消息需要借助Spring的`编程式事务`或者`声明式事务`。编程式事务一般依赖`于TransactionTemplate`，而声明式事务依托于AOP模块，依赖于注解`@Transactional`。

接着需要自定义一个事务消息功能模块，新增一个事务消息记录表（其实就是`本地消息`表），用于保存每一条需要发送的消息记录。事务消息功能模块的主要功能是：

* 保存消息记录。
* 推送消息到RabbitMQ服务端。
* 消息记录的查询、补偿推送等等。

## **事务执行的逻辑单元**

在事务执行的逻辑单元里面，需要进行待推送的事务消息记录的保存，也就是：**本地（业务）逻辑和事务消息记录保存操作绑定在同一个事务**。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/11/640-20200711182849249-182849.png)

发送消息到RabbitMQ服务端这一步需要延后到**事务提交之后**，这样才能保证事务提交成功和消息成功发送到RabbitMQ服务端这两个操作是一致的。

为了把保存**待发送的事务消息**和**发送消息到RabbitMQ**两个动作从使用者感知角度合并为一个动作，这里需要用到Spring特有的事务同步器`TransactionSynchronization`，这里分析一下事务同步器的主要方法的回调位置，主要参考`AbstractPlatformTransactionManager#commit()`或者`AbstractPlatformTransactionManager#processCommit()`方法：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/11/640-20200711182849893-182849.png)

上图仅仅演示了事务正确提交的场景（不包含异常的场景）。这里可以明确知道，事务同步器TransactionSynchronization的afterCommit\(\)和afterCompletion\(int status\)方法都在真正的事务提交点AbstractPlatformTransactionManager\#doCommit\(\)之后回调，因此可以选用这两个方法其中之一用于执行推送消息到RabbitMQ服务端，整体的伪代码如下：

```text
@Transactional
public Dto businessMethod(){
    business transaction code block ...
    // 保存事务消息
    [saveTransactionMessageRecord()]
    // 注册事务同步器 - 在afterCommit()方法中推送消息到RabbitMQ
    [register TransactionSynchronization,send message in method afterCommit()]
    business transaction code block ...
}
```

上面伪代码中，**保存事务消息**和**注册事务同步器**两个步骤可以安插在事务方法中的任意位置，也就是说与执行顺序无关。

## **事务消息的补偿**

虽然之前提到笔者建议下游服务自理自身服务消费异常的场景，但是有些时候迫于无奈还是需要上游把对应的消息重新推送，这个算是特殊的场景。

另外还有一个场景需要考虑：事务提交之后触发事务同步器`TransactionSynchronization`的`afterCommit()`方法失败。这是一个低概率的场景，但是在生产中一定会出现，一个比较典型的原因就是：**事务提交完成后尚未来得及触发TransactionSynchronization\#afterCommit\(\)方法进行推送服务实例就被重启**。

如下图所示：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/11/640-20200711182850245-182850.png)

为了统一处理补偿推送的问题，使用了有限状态判断消息是否已经推送成功：

* 在事务方法内，保存事务消息的时候，标记消息记录推送状态为处理中。
* 事务同步器接口TransactionSynchronization的afterCommit\(\)方法的实现中，推送对应的消息到RabbitMQ，然后更变事务消息记录的状态为推送成功。

还有一种极为特殊的情况是RabbitMQ服务端本身出现故障导致消息推送异常，这种情况下需要进行重试（补偿推送），**经验证明短时间内的反复重试是没有意义的**，故障的服务一般不会瞬时恢复，所以可以考虑使用**指数退避算法**进行重试，同时需要限制最大重试次数。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/11/640-20200711182850358-182850.png)

指数值、间隔值和最大重试次数上限需要根据实际情况设定，否则容易出现消息延时过大或者重试过于频繁等问题。

## **方案实施**

引入核心依赖：

```text
<properties>
    <spring.boot.version>2.2.4.RELEASE</spring.boot.version>
    <redisson.version>3.12.1</redisson.version>
    <mysql.connector.version>5.1.48</mysql.connector.version>
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring.boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.connector.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <dependency>
        <groupId>org.redisson</groupId>
        <artifactId>redisson</artifactId>
        <version>${redisson.version}</version>
    </dependency>
</dependencies>
```

spring-boot-starter-jdbc、mysql-connector-java和spring-boot-starter-aop是MySQL事务相关，而spring-boot-starter-amqp是RabbitMQ客户端的封装，redisson主要使用其分布式锁，用于补偿定时任务的加锁执行（以防止服务多个节点并发执行补偿推送）。

## **表设计**

事务消息模块主要涉及两张表，以MySQL为例，建表DDL如下：

```text
CREATE TABLE `t_transactional_message`
(
    id                  BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    create_time         DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP,
    edit_time           DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    creator             VARCHAR(20)     NOT NULL DEFAULT 'admin',
    editor              VARCHAR(20)     NOT NULL DEFAULT 'admin',
    deleted             TINYINT         NOT NULL DEFAULT 0,
    current_retry_times TINYINT         NOT NULL DEFAULT 0 COMMENT '当前重试次数',
    max_retry_times     TINYINT         NOT NULL DEFAULT 5 COMMENT '最大重试次数',
    queue_name          VARCHAR(255)    NOT NULL COMMENT '队列名',
    exchange_name       VARCHAR(255)    NOT NULL COMMENT '交换器名',
    exchange_type       VARCHAR(8)      NOT NULL COMMENT '交换类型',
    routing_key         VARCHAR(255) COMMENT '路由键',
    business_module     VARCHAR(32)     NOT NULL COMMENT '业务模块',
    business_key        VARCHAR(255)    NOT NULL COMMENT '业务键',
    next_schedule_time  DATETIME        NOT NULL COMMENT '下一次调度时间',
    message_status      TINYINT         NOT NULL DEFAULT 0 COMMENT '消息状态',
    init_backoff        BIGINT UNSIGNED NOT NULL DEFAULT 10 COMMENT '退避初始化值,单位为秒',
    backoff_factor      TINYINT         NOT NULL DEFAULT 2 COMMENT '退避因子(也就是指数)',
    INDEX idx_queue_name (queue_name),
    INDEX idx_create_time (create_time),
    INDEX idx_next_schedule_time (next_schedule_time),
    INDEX idx_business_key (business_key)
) COMMENT '事务消息表';

CREATE TABLE `t_transactional_message_content`
(
    id         BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    message_id BIGINT UNSIGNED NOT NULL COMMENT '事务消息记录ID',
    content    TEXT COMMENT '消息内容'
) COMMENT '事务消息内容表';
```

因为此模块有可能扩展出一个后台管理模块，所以要把消息的管理和状态相关字段和大体积的消息内容分别存放在两个表，从而避免大批量查询消息记录的时候MySQL服务IO使用率过高的问题（这是和上一个公司的DBA团队商讨后得到的一个比较合理的方案）。预留了两个业务字段`business_module`和`business_key`用于标识业务模块和业务键（一般是唯一识别号，例如订单号）。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/11/640-20200711182850439-182850.jpg)

一般情况下，如果服务通过配置自行提前声明队列和交换器的绑定关系，那么发送RabbitMQ消息的时候其实只依赖于exchangeName和routingKey两个字段（header类型的交换器是特殊的，也比较少用，这里暂时不用考虑），考虑到服务可能会遗漏声明操作，发送消息的时候会基于队列进行首次绑定声明并且缓存相关的信息（RabbitMQ中的队列-交换器绑定声明只要每次声明绑定关系的参数一致，则不会抛出异常）。

## **方案代码设计**

下面的方案设计描述中，暂时忽略了消息事务管理后台的API设计，这些可以在后期补充。

> 定义模型实体类TransactionalMessage和TransactionalMessageContent：

```text
@Data
public class TransactionalMessage {

    private Long id;
    private LocalDateTime createTime;
    private LocalDateTime editTime;
    private String creator;
    private String editor;
    private Integer deleted;
    private Integer currentRetryTimes;
    private Integer maxRetryTimes;
    private String queueName;
    private String exchangeName;
    private String exchangeType;
    private String routingKey;
    private String businessModule;
    private String businessKey;
    private LocalDateTime nextScheduleTime;
    private Integer messageStatus;
    private Long initBackoff;
    private Integer backoffFactor;
}

@Data
public class TransactionalMessageContent {

    private Long id;
    private Long messageId;
    private String content;
}
```

> 然后定义dao接口（这里暂时不展开实现的细节代码，存储使用MySQL，如果要替换为其他类型的数据库，只需要使用不同的实现即可）：

```text
public interface TransactionalMessageDao {

    void insertSelective(TransactionalMessage record);

    void updateStatusSelective(TransactionalMessage record);

    List<TransactionalMessage> queryPendingCompensationRecords(LocalDateTime minScheduleTime,
                                                               LocalDateTime maxScheduleTime,
                                                               int limit);
}

public interface TransactionalMessageContentDao {

    void insert(TransactionalMessageContent record);

    List<TransactionalMessageContent> queryByMessageIds(String messageIds);
}
```

> 接着定义事务消息服务接口TransactionalMessageService：

```text
// 对外提供的服务类接口
public interface TransactionalMessageService {

    void sendTransactionalMessage(Destination destination, TxMessage message);
}


@Getter
@RequiredArgsConstructor
public enum ExchangeType {

    FANOUT("fanout"),

    DIRECT("direct"),

    TOPIC("topic"),

    DEFAULT(""),

    ;

    private final String type;
}

// 发送消息的目的地
public interface Destination {

    ExchangeType exchangeType();

    String queueName();

    String exchangeName();

    String routingKey();
}

@Builder
public class DefaultDestination implements Destination {

    private ExchangeType exchangeType;
    private String queueName;
    private String exchangeName;
    private String routingKey;

    @Override
    public ExchangeType exchangeType() {
        return exchangeType;
    }

    @Override
    public String queueName() {
        return queueName;
    }

    @Override
    public String exchangeName() {
        return exchangeName;
    }

    @Override
    public String routingKey() {
        return routingKey;
    }
}

// 事务消息
public interface TxMessage {

    String businessModule();

    String businessKey();

    String content();
}

@Builder
public class DefaultTxMessage implements TxMessage {

    private String businessModule;
    private String businessKey;
    private String content;

    @Override
    public String businessModule() {
        return businessModule;
    }

    @Override
    public String businessKey() {
        return businessKey;
    }

    @Override
    public String content() {
        return content;
    }
}

// 消息状态
@RequiredArgsConstructor
public enum TxMessageStatus {

    /**
     * 成功
     */
    SUCCESS(1),

    /**
     * 待处理
     */
    PENDING(0),

    /**
     * 处理失败
     */
    FAIL(-1),

    ;

    private final Integer status;
}
```

> TransactionalMessageService的实现类是事务消息的核心功能实现，代码如下：

```text
@Slf4j
@Service
@RequiredArgsConstructor
public class RabbitTransactionalMessageService implements TransactionalMessageService {

    private final AmqpAdmin amqpAdmin;
    private final TransactionalMessageManagementService managementService;

    private static final ConcurrentMap<String, Boolean> QUEUE_ALREADY_DECLARE = new ConcurrentHashMap<>();

    @Override
    public void sendTransactionalMessage(Destination destination, TxMessage message) {
        String queueName = destination.queueName();
        String exchangeName = destination.exchangeName();
        String routingKey = destination.routingKey();
        ExchangeType exchangeType = destination.exchangeType();
        // 原子性的预声明
        QUEUE_ALREADY_DECLARE.computeIfAbsent(queueName, k -> {
            Queue queue = new Queue(queueName);
            amqpAdmin.declareQueue(queue);
            Exchange exchange = new CustomExchange(exchangeName, exchangeType.getType());
            amqpAdmin.declareExchange(exchange);
            Binding binding = BindingBuilder.bind(queue).to(exchange).with(routingKey).noargs();
            amqpAdmin.declareBinding(binding);
            return true;
        });
        TransactionalMessage record = new TransactionalMessage();
        record.setQueueName(queueName);
        record.setExchangeName(exchangeName);
        record.setExchangeType(exchangeType.getType());
        record.setRoutingKey(routingKey);
        record.setBusinessModule(message.businessModule());
        record.setBusinessKey(message.businessKey());
        String content = message.content();
        // 保存事务消息记录
        managementService.saveTransactionalMessageRecord(record, content);
        // 注册事务同步器
        TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronizationAdapter() {
            @Override
            public void afterCommit() {
                managementService.sendMessageSync(record, content);
            }
        });
    }
}
```

> 消息记录状态和内容持久化的管理统一放在TransactionalMessageManagementService中：

```text
@Slf4j
@RequiredArgsConstructor
@Service
public class TransactionalMessageManagementService {

    private final TransactionalMessageDao messageDao;
    private final TransactionalMessageContentDao contentDao;
    private final RabbitTemplate rabbitTemplate;

    private static final LocalDateTime END = LocalDateTime.of(2999, 1, 1, 0, 0, 0);
    private static final long DEFAULT_INIT_BACKOFF = 10L;
    private static final int DEFAULT_BACKOFF_FACTOR = 2;
    private static final int DEFAULT_MAX_RETRY_TIMES = 5;
    private static final int LIMIT = 100;

    public void saveTransactionalMessageRecord(TransactionalMessage record, String content) {
        record.setMessageStatus(TxMessageStatus.PENDING.getStatus());
        record.setNextScheduleTime(calculateNextScheduleTime(LocalDateTime.now(), DEFAULT_INIT_BACKOFF,
                DEFAULT_BACKOFF_FACTOR, 0));
        record.setCurrentRetryTimes(0);
        record.setInitBackoff(DEFAULT_INIT_BACKOFF);
        record.setBackoffFactor(DEFAULT_BACKOFF_FACTOR);
        record.setMaxRetryTimes(DEFAULT_MAX_RETRY_TIMES);
        messageDao.insertSelective(record);
        TransactionalMessageContent messageContent = new TransactionalMessageContent();
        messageContent.setContent(content);
        messageContent.setMessageId(record.getId());
        contentDao.insert(messageContent);
    }

    public void sendMessageSync(TransactionalMessage record, String content) {
        try {
            rabbitTemplate.convertAndSend(record.getExchangeName(), record.getRoutingKey(), content);
            if (log.isDebugEnabled()) {
                log.debug("发送消息成功,目标队列:{},消息内容:{}", record.getQueueName(), content);
            }
            // 标记成功
            markSuccess(record);
        } catch (Exception e) {
            // 标记失败
            markFail(record, e);
        }
    }

    private void markSuccess(TransactionalMessage record) {
        // 标记下一次执行时间为最大值
        record.setNextScheduleTime(END);
        record.setCurrentRetryTimes(record.getCurrentRetryTimes().compareTo(record.getMaxRetryTimes()) >= 0 ?
                record.getMaxRetryTimes() : record.getCurrentRetryTimes() + 1);
        record.setMessageStatus(TxMessageStatus.SUCCESS.getStatus());
        record.setEditTime(LocalDateTime.now());
        messageDao.updateStatusSelective(record);
    }

    private void markFail(TransactionalMessage record, Exception e) {
        log.error("发送消息失败,目标队列:{}", record.getQueueName(), e);
        record.setCurrentRetryTimes(record.getCurrentRetryTimes().compareTo(record.getMaxRetryTimes()) >= 0 ?
                record.getMaxRetryTimes() : record.getCurrentRetryTimes() + 1);
        // 计算下一次的执行时间
        LocalDateTime nextScheduleTime = calculateNextScheduleTime(
                record.getNextScheduleTime(),
                record.getInitBackoff(),
                record.getBackoffFactor(),
                record.getCurrentRetryTimes()
        );
        record.setNextScheduleTime(nextScheduleTime);
        record.setMessageStatus(TxMessageStatus.FAIL.getStatus());
        record.setEditTime(LocalDateTime.now());
        messageDao.updateStatusSelective(record);
    }

    /**
     * 计算下一次执行时间
     *
     * @param base          基础时间
     * @param initBackoff   退避基准值
     * @param backoffFactor 退避指数
     * @param round         轮数
     * @return LocalDateTime
     */
    private LocalDateTime calculateNextScheduleTime(LocalDateTime base,
                                                    long initBackoff,
                                                    long backoffFactor,
                                                    long round) {
        double delta = initBackoff * Math.pow(backoffFactor, round);
        return base.plusSeconds((long) delta);
    }

    /**
     * 推送补偿 - 里面的参数应该根据实际场景定制
     */
    public void processPendingCompensationRecords() {
        // 时间的右值为当前时间减去退避初始值，这里预防把刚保存的消息也推送了
        LocalDateTime max = LocalDateTime.now().plusSeconds(-DEFAULT_INIT_BACKOFF);
        // 时间的左值为右值减去1小时
        LocalDateTime min = max.plusHours(-1);
        Map<Long, TransactionalMessage> collect = messageDao.queryPendingCompensationRecords(min, max, LIMIT)
                .stream()
                .collect(Collectors.toMap(TransactionalMessage::getId, x -> x));
        if (!collect.isEmpty()) {
            StringJoiner joiner = new StringJoiner(",", "(", ")");
            collect.keySet().forEach(x -> joiner.add(x.toString()));
            contentDao.queryByMessageIds(joiner.toString())
                    .forEach(item -> {
                        TransactionalMessage message = collect.get(item.getMessageId());
                        sendMessageSync(message, item.getContent());
                    });
        }
    }
}
```

> 这里有一点尚待优化：更新事务消息记录状态的方法可以优化为批量更新，在limit比较大的时候，批量更新的效率会更高。最后是定时任务的配置类：

```text
@Slf4j
@RequiredArgsConstructor
@Configuration
@EnableScheduling
public class ScheduleJobAutoConfiguration {

    private final TransactionalMessageManagementService managementService;

    /**
     * 这里用的是本地的Redis,实际上要做成配置
     */
    private final RedissonClient redisson = Redisson.create();

    @Scheduled(fixedDelay = 10000)
    public void transactionalMessageCompensationTask() throws Exception {
        RLock lock = redisson.getLock("transactionalMessageCompensationTask");
        // 等待时间5秒,预期300秒执行完毕,这两个值需要按照实际场景定制
        boolean tryLock = lock.tryLock(5, 300, TimeUnit.SECONDS);
        if (tryLock) {
            try {
                long start = System.currentTimeMillis();
                log.info("开始执行事务消息推送补偿定时任务...");
                managementService.processPendingCompensationRecords();
                long end = System.currentTimeMillis();
                long delta = end - start;
                // 以防锁过早释放
                if (delta < 5000) {
                    Thread.sleep(5000 - delta);
                }
                log.info("执行事务消息推送补偿定时任务完毕,耗时:{} ms...", end - start);
            } finally {
                lock.unlock();
            }
        }
    }
}
```

> 基本代码编写完，整个项目的结构如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/11/640-20200711182850743-182850.png)

> 最后添加两个测试类：

```text
@RequiredArgsConstructor
@Component
public class MockBusinessRunner implements CommandLineRunner {

    private final MockBusinessService mockBusinessService;

    @Override
    public void run(String... args) throws Exception {
        mockBusinessService.saveOrder();
    }
}

@Slf4j
@RequiredArgsConstructor
@Service
public class MockBusinessService {

    private final JdbcTemplate jdbcTemplate;
    private final TransactionalMessageService transactionalMessageService;
    private final ObjectMapper objectMapper;

    @Transactional(rollbackFor = Exception.class)
    public void saveOrder() throws Exception {
        String orderId = UUID.randomUUID().toString();
        BigDecimal amount = BigDecimal.valueOf(100L);
        Map<String, Object> message = new HashMap<>();
        message.put("orderId", orderId);
        message.put("amount", amount);
        jdbcTemplate.update("INSERT INTO t_order(order_id,amount) VALUES (?,?)", p -> {
            p.setString(1, orderId);
            p.setBigDecimal(2, amount);
        });
        String content = objectMapper.writeValueAsString(message);
        transactionalMessageService.sendTransactionalMessage(
                DefaultDestination.builder()
                        .exchangeName("tm.test.exchange")
                        .queueName("tm.test.queue")
                        .routingKey("tm.test.key")
                        .exchangeType(ExchangeType.DIRECT)
                        .build(),
                DefaultTxMessage.builder()
                        .businessKey(orderId)
                        .businessModule("SAVE_ORDER")
                        .content(content)
                        .build()
        );
        log.info("保存订单:{}成功...", orderId);
    }
}
```

> 某次测试结果如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/11/640-20200711182850949-182851.png)

模拟订单数据成功保存，而且RabbitMQ消息在事务成功提交后正常发送到RabbitMQ服务端中，如RabbitMQ控制台数据所示。

## **小结**

事务消息模块的设计仅仅是使异步消息推送这个功能实现趋向于完备，其实一个合理的异步消息交互系统，一定会提供同步查询接口，这一点是基于异步消息没有回调或者没有响应的特性导致的。

一般而言，一个系统的吞吐量和系统的异步化处理占比成正相关（这一点可以参考Amdahl's Law），所以在系统架构设计实际中应该尽可能使用异步交互，提高系统吞吐量同时减少同步阻塞带来的无谓等待。事务消息模块可以扩展出一个后台管理，甚至可以配合Micrometer、Prometheus和Grafana体系做实时数据监控。

