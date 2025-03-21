# 两套详细的设计方案，解决头疼的掉单问题

[https://mp.weixin.qq.com/s/YIXrJJIsthQVSLLZdm5mzA](https://mp.weixin.qq.com/s/YIXrJJIsthQVSLLZdm5mzA)

Hello，大家好，我是楼下小黑哥~

好久没写支付相关的文章了，今天继续从事老本行~

上次在文章[钱被扣走了，但是订单却未成功！支付掉单异常最全解决方案](https://mp.weixin.qq.com/s?__biz=MzIzMTgwODgyMw==&mid=2247485215&idx=1&sn=6c8c37a118763275b3ca4f5c77ca7575&scene=21#wechat_redirect)提到，支付过程会出现**「掉单、卡单」**的情况，这种情况对于用户来讲，体验非常差，明明自己付了钱，扣了款，但是订单却未成功。

上篇文章我们简单说了下这种情况通常采用异步补偿方式，这次小黑哥就结合生产实际碰到的情况，给出两种详细设计的方案：

* 定时轮询补偿方案
* 延迟消息补偿方案

大家可以根据自己系统的实际情况，选择性参考。

**「当然了，以下设计方案可能并不完美，如果各位读者还有其他解决方案，欢迎留言指出，一起讨论，一起成长~」**

## 定时轮询补偿方案

### 整体流程

这个方案主要采用定时任务，批量查询掉单记录，从而驱动查询具体支付支付结果，然后更新内部订单。

整体方案流程图如下：

定时任务补偿

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/01/06/640-113405.jpg)

前三步流程没什么好说的，正常的支付流程，咱们针对后面几步具体详细说下。

第三步调用支付通道之后，如果支付通道端返回**「支付受理成功或者支付处理中」**，我们就需要调用第四步，将这类订单插入掉单表。

如果支付直接成功了，那就正常流程返回即可。

> 复习一下，网关类支付，比如支付宝、微信支付、网银支付，这种支付模式，支付通道仅仅返回支付受理成功，具体支付结果需要接收支付通道端的支付通知，这类支付我们将其称为异步支付。
>
> 相应的还有同步支付，比如银行卡支付，微信、支付宝代扣类支付，这类支付，同步就能返回支付结果。

第五步，补单应用将会定时查询数据库，批量查询掉单记录。

第六步，补单应用使用线程池，多线程异步的方式发起掉单查询。

第七步，调用支付通道支付查询接口。

重点来了，如果第七步支付结果查询为以下状态：

* **「支付结果为扣款成功」**
* **「支付结果为明确失败」**
* **「掉单记录查询达到最大次数」**

**「第八步就会删除掉单记录。」**

最后，如果掉单查询依旧还是处理中，那么经过一定的延时之后，重复第五步，再次重新掉单补偿，直到成功或者查询到达最大次数。

### 相关问题

**「为什么需要新建一张掉单表？不能直接使用支付订单表，查询未成功的订单吗?」**

这个问题，实际上确实可以直接使用的支付订单表，然后批量查询当天未成功的订单，补单程序发起支付查询。

那为什么需要新建一张掉单表？

主要是因为数据库查询效率问题，因为支付订单表每天都会大量记录新增，随着时间，这张表记录将会越来越多，越来越大。

**「支付记录越多，批量范围查询效率就会变低，查询速度将会变慢。」**

所以为了查询效率，新建一张掉单表。

这张表里仅记录支付未成功的订单，所以数据量就会很小，那么查询效率就会很高。

另外，掉单表里的记录，不会被永久保存，只是临时性。当支付结果查询成功，或者支付结果明确失败，再或者查询次数到达规定最大次数，就会删除掉单记录。

**「这就是第八步为什么需要删除掉单表的原因。」**

如果需要保存每次掉单表查询详情，那么这里建议再新增一张掉单查询记录表，保存每一次的查询记录。

针对这个方案，如果还有其他问题，欢迎留言。

### 方案优缺点

定时轮询补偿方案，最大的优点可能就是系统架构方案比较简单，比较容易实施。

那么这个方案的缺点主要在于**「定时任务」**上。

定时任务轮询方案天然会存在以下不足：

1. **「轮询效率稍低」**
2. 每次查询数据库，已经被执行过记录，仍然会被扫描（补单程序将会根据一定策略决定是否发起支付通道查询），有**「重复计算」**的嫌疑
3. **「时效性不够好」**，如果每小时轮询一次，最差的情况下，时间误差会达到1小时
4. 如果为了解决时效性问题，增加定时任务查询效率，那么 1 中查询效率跟 2 的重复计算问题将会更加明显。

## 延迟消息补偿方案

下面介绍另外一种掉单补偿方案，延迟消息补偿方案，这个方案整体流程与定时任务方案类似，最大区别可能在于，从一种**「拉模式」**变成一种**「推模式」**。

整体方案流程图如下：

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/01/06/640-20210106113405603-113405.jpg)

这个方案主要流程跟定时方案类似，主要区别在于第四步，第五步，第八步。

第四步的流程从插入掉单表变更为往**「延迟队列发送掉单消息」**。

第五步，补单程序接收掉单消息，然后触发支付掉单查询。

第八步，如果第七步支付结果查询为以下状态：

* 支付结果为扣款成功
* 支付结果为明确失败
* 掉单记录查询达到最大次数

补单程序将会告知延迟队列消费成功，延迟队列将会删除这条掉单消息。

其他状态将会告知消费失效，延迟队列将会在一定延时之后，再次发送掉单消息，然后继续重复第五步。

### 延迟队列

这里的延迟队列需要自己实现，复杂度还是比较高的，这里给大家推荐几种实现方案：

第一种，基于 **「Redis SortedSet」** 实现延迟队列。可以参考一下有赞的实现方案[https://tech.youzan.com/queuing\_delay/](https://tech.youzan.com/queuing_delay/)

第二种，基于时间轮算法\(**「TimingWheel」**\)实现延迟队列，具体可以参考 Kafka 延时队列。

第三种，基于 **「RocketMQ」** 延迟消息。

前两种方案说起来还需要再开发，所以还是比较复杂的。

这里重点说下第三种方案，该方案是 **「RocketMQ」** 已经支持的特性，开箱即用，使用起来还是比较简单的。

RocketMQ 延迟消息支持 18 个等级，分别如下：

```text
1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
```

消息发送方可以通过以下方式指定延迟等级，对应上方的延迟时间。

```text
Message#setDelayTimeLevel
```

消息消费方，如果消费失败，默认将会在消息发送方的的延迟等级基础上加 1。如果消息消费方需要指定其他的延迟等级，可以使用如下方式：

```text
ConsumeConcurrentlyContext#setDelayLevelWhenNextConsume
```

RocketMQ 延迟消息，支持的特性还是比较基础、简单，不支持自定义延迟时间。不过对于掉单补偿的这个场景刚好够用，但是如果需要自定义延迟的，那还是得采用其他的方案。

### 方案优缺点

延迟消息的方案相对于定时轮询方案来讲：

* 无需再查询全部订单，效率高
* 时效性较好

不过延迟消息这种方案，需要基于**「延迟队列」**，实现起来比较复杂，目前开源实现也比较少。

## 小结

支付掉单、卡单是支付过程中经常会碰到的事，我们可以采用异步补偿的方案，解决该问题。

异步补偿方案可以采用如下两种：

* 定时轮询补偿方案
* 延迟消息补偿方案

定时轮询补偿方案实现起来比较简单，但是时效性稍差。

而延迟消息补偿方案总体来说比较优秀，但是实现起来比较复杂。如果没有自定义的延迟时间的需求，可以直接采用 RocketMQ 延迟消息，简单快捷。

另外**「延迟队列」**使用场景还是比较多，不仅仅能用在掉单补偿上，还可以用于支付关单等场景。所以有能力开发的团队，可以开发一个通用的延迟队列、

好了，今天的文章就到这里了。

我是楼下小黑哥，下篇文章再见，886~

