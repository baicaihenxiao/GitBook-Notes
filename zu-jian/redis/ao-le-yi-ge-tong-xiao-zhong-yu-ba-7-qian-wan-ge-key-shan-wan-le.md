# 熬了一个通宵，终于把7千万个Key删完了

[https://mp.weixin.qq.com/s/mG\_47Jklujtp1b1mxaYC2w](https://mp.weixin.qq.com/s/mG_47Jklujtp1b1mxaYC2w)

### 前言

由于有一条业务线不理想，高层决定下架业务。对于我们技术团队而言，其对应的所有服务器资源和其他相关资源都要释放。释放了8台应用服务器；1台es服务器；删除分布式定时任务中心相关的业务任务；备份并删除MySQL数据库；删除Redis中相关的业务缓存数据。CTO指名点姓让我带头冲锋，才扣了我绩效……好吧，冲~ 其他都还好，不多时就解决了。唯独这删除Redis中的数据，害得我又熬了一个通宵，真是折煞我也！

### 难点分析

#### 共用Redis服务集群

由于这条业务线的数据在Redis大概在3G左右，完全没必要单独建一个Redis服务集群，本着能节约就节约的态度，当初就决定和其他项目共享一个集群（这个集群配置：16个节点，128G内存，还算豪华吧~）集群配置如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/14/1-115540.jpg)

在这种共用集群的情况下，导致无法简单粗暴的释放。因此只能选择删除Key的方式。

#### Key命名不规范

要删除Key，首先就要精准的定位出哪些Key需要删除，如果勿删Key，会影响到其他服务正常运转！如果Key本身设置了过期时间，但有些数据需是持久化的。然而那该死的项目经理一直催项目进度，导致开发人员在开发过程中很多地方都没有设计到位，比如Redis Key散落在项目代码的每个角落；比如命名不是很规范。真不知道是怎么review代码！哦，想必是没有时间review，那该死的项目经理…… 我随便截个支付服务中的Key命名：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/14/1-20200814115540775-115540.jpg)

怎么样？是不是觉得我们开发人员写的代码很low~别笑，在实际工作中，还有比这更low的！希望你别遇到，不然真的很痛苦~

### 解决思路

经过以上的分析，我们简单归纳如下：

* 我们真正关心的是那些未设置过期时间的Key
* 不能误删除Key，否则下个月绩效也没了
* 由于Key的命名及使用及其不规范，导致Key的定位难度很大

看来，通过scan命令扫描匹配Key的方式行不通了。只能通过人肉搜索了~ 幸而Idea的搜索大法好，这个项目中使用的是spring-boot-starter-data-redis.因此我通过搜索RedisTemplate和StringRedisTemplate定位所有操作redis的代码，具体步骤如下： 1.通过这些代码统计出Key的前缀并录入到文本中； 2.通过python脚本把载入文中中的的Key并在后面加上“\*”通配符； 3.通过python脚本通过scan命令扫描出这些key； 4.为了便于检查，我们并没有直接使用del命令删除key，在删除key之前，先通过debug object key的方式得到其序列化的长度，再执行删除并返回序列化长度。这样，我们就可以统计出所有key的序列化长度来得到我们释放的空间大小。 关键代码如下：

```text
def get_key(rdbConn,start):
    try:
    keys_list = rdbConn.scan(start,count=2000)
    return keys_list
    except Exception,e:
    print e

''' Redis DEBUG OBJECT command got key info '''
def get_key_info(rdbConn,keyName):
    try:
    rpiple = rdbConn.pipeline()
    rpiple.type(keyName)
    rpiple.debug_object(keyName)
    rpiple.ttl(keyName)
    key_info_list = rpiple.execute()
    return key_info_list
    except Exception,e:
    print "INFO : ",e

def redis_key_static(key_info_list):
    keyType = key_info_list[0]
    keySize = key_info_list[1]['serializedlength']
    keyTtl = key_info_list[2]
    key_size_static(keyType,keySize,keyTtl)
复制代码
```

通过以上方式，能够统计出究竟释放了多少内存了。 由于这个集群是有这么接近7千万个key：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/14/1-20200814115540972-115541.jpg)

因此，等到了第二天天亮，我睡眼朦胧的看了一下，终于删除完毕了，时间07:13...早高峰即将来临……

### 知耻而后勇

从来没有经历过因业务下线而清除资源的经验。这次事情真心让我觉得细微之处见真功夫的道理。如果一开始我们就能够遵循开发规范来使用和设计redis key，也不至于浪费这么多时间。为了让key的命名和使用更加规范，以及今后避免再次遇到这种情况，下午睡醒之后，我就在redis公共组件库里面添加了一个配置和自定义了key序列化，代码如下：

```text
@ConfigurationProperties(prefix = "spring.redis.prefix")
public class RedisKeyPrefixProperties {
    private Boolean enable = Boolean.TRUE;
    private String key;
    public Boolean getEnable() {
        return enable;
    }
    public void setEnable(Boolean enable) {
        this.enable = enable;
    }
    public String getKey() {
        return key;
    }
    public void setKey(String key) {
        this.key = key;
    }
}
复制代码
/**
 * @desc 对字符串序列化新增前缀
 * @author create by liming sun on 2020-07-21 14:09:51
 */
public class PrefixStringKeySerializer extends StringRedisSerializer {
    private Charset charset = StandardCharsets.UTF_8;
    private RedisKeyPrefixProperties prefix;

    public PrefixStringKeySerializer(RedisKeyPrefixProperties prefix) {
        super();
        this.prefix = prefix;
    }

    @Override
    public String deserialize(@Nullable byte[] bytes) {
        String saveKey = new String(bytes, charset);
        if (prefix.getEnable() != null && prefix.getEnable()) {
            String prefixKey = spliceKey(prefix.getKey());
            int indexOf = saveKey.indexOf(prefixKey);
            if (indexOf > 0) {
                saveKey = saveKey.substring(indexOf);
            }
        }
        return (saveKey.getBytes() == null ? null : saveKey);
    }

    @Override
    public byte[] serialize(@Nullable String key) {
        if (prefix.getEnable() != null && prefix.getEnable()) {
            key = spliceKey(prefix.getKey()) + key;
        }
        return (key == null ? null : key.getBytes(charset));
    }

    private String spliceKey(String prefixKey) {
        if (StringUtils.isNotBlank(prefixKey) && !prefixKey.endsWith(":")) {
            prefixKey = prefixKey + "::";
        }
        return prefixKey;
    }
}
复制代码
```

#### 使用效果

为了避免再次发生这种工作低效而又不得不做的事情，我们在开发规范中规定，新项目中redis的使用必须设置此配置，前缀就设置为：项目编号。另外，一个模块中的key必须统一定义在二方库的RedisKeyConstant类中。配置如下：

```text
spring: 
    redis: 
        prefix:
            enable: true
            key: E00P01
复制代码
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(redisConnectionFactory);
    // 支持key前缀设置的key Serializer
    redisTemplate.setKeySerializer(new PrefixStringKeySerializer());
    redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
    return redisTemplate;
}
复制代码
```

通过以上方式，我们至少可以从项目维度来区分出key，避免了多个项目之间共用同一个集群时而导致重复key的问题。从项目维度对key进行了划分。更方便管理和运维。如果对于key的管理粒度要求更细，我们甚至可以细化到具体业务维度。我们在测试环境进行了压测，增加key前缀对redis性能几乎没有影响。性能方面能接受。

### 总结

通过本次事情，我发现对于大多数开发者而言，差距其实不在于智力，而是在于态度。比如这次事件暴露出来的问题：大家都知道要遵循开发规范，然而到了真正“打仗”的时候，负责这个项目的开发者却没有几个人能始终如一的做好这些细微之事。另外，reviewer的工作其实是极其重要的，他就像那“纪检委”，如果“纪检委”都放水睁一只眼闭一只眼，那麻烦可就大了！千里之提，毁于日常的点滴松懈啊~~~ 经过这次事件之后，如果上天再给一次这样的机会，我一定会对项目经理说：接着奏乐，接着舞~

作者：浪漫先生 链接：[https://juejin.im/post/6854573215726075917](https://juejin.im/post/6854573215726075917) 来源：掘金 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

