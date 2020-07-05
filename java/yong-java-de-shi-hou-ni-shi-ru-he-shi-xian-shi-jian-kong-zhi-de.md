# 用Java的时候，你是如何实现时间控制的？

{% embed url="https://mp.weixin.qq.com/s/\_rMMBpYe9jbaypbhYotGWg" %}



作者 \| Yrion

来源 \| rrd.me/gCQHp

前言：需求是这样的，在与第三方对接过程中，对方提供了token进行时效性验证，过一段时间token就会失效.后台有定时任务在获取，但是偶尔会出现token失效，这是因为在获取的时候，定时任务正在跑，可能正在获取最新的token中，这个时候如何过一段时间\(比如800毫秒之后\)再请求呢？小王仰望天空45度，思考起来了。。。

**一：时间控制的几种方案**

1.1: 从线程方面解决

最简单粗暴的一种实现方案：Thread.sleep\(800\)，但是很快就被小王给pass掉了。为什么呢？虽然这种方式可以，但是存在一个隐患，如果在多线程环境下，线程很容易被interrupt,这样代码就会抛出异常，这样线程就会挂起，导致整个线程异常结束。实在是不够优雅，违背了我们设计的初衷。

1.2:使用Timer

查阅了jdk，我发现有个实现定时的类，使用它是可以的，在jdk中提供了定时器类，这个类的主要作用就是控制一定的时间来简单的定时执行某个任务。有点简单的elasticJob的设计味道。接下来看一下，用timmer如何实现延时。。有点惊喜，我们来写一个最简单的例子来看一下如何实现定时任务：

```text
public class TimmerTest {
   /**
     * 测试方法
     */
    public void test() {
        Timer timer = new Timer();
        timer.schedule(new MyTask(), 800);
    }
    
    public class MyTask extends TimerTask {
        
        /**
         * 运行方法
         */
        @Override
        public void run() {
            System.out.println("输出");
        }
    }
}
```

```text
<dependency>
  <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
      <exclusions>
        <exclusion>
          <groupId>io.lettuce</groupId>
          <artifactId>lettuce-core</artifactId>
        </exclusion>
      </exclusions>
</dependency>
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
</dependency>
```

```text
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
​
@Configuration
public class RedisConfig {
    
    @Autowired
    private RedisTemplate redisTemplate;
    
    /**
     * redisTemplate实例化
     *
     * @return
     */
    @Bean
    public RedisTemplate redisTemplateInit() {
        //设置序列化Key的实例化对象
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        //设置序列化Value的实例化对象
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return redisTemplate;
    }
    
}
```

```text
@Component
public class RedisManager {
    
    private static final Logger LOGGER = LoggerFactory.getLogger(RedisManager.class);
    
    @Autowired
    private RedisTemplate redisTemplate;
    
    /**
     * 设置对象
     *
     * @param key key
     * @param value value值
     * @param <T> 返回值泛型
     * @return 正确的值：<T> 错误的值：null
     */
    @SuppressWarnings("unchecked")
    public <T> ValueOperations<String, T> setObject(final String key, final T value) {
        final ValueOperations<String, T> operation = redisTemplate.opsForValue();
        operation.set(key, value);
        return operation;
    }
    
    /**
     * 设置对象及失效时间 (单位：秒)
     *
     * @param key key
     * @param value value值
     * @param <T> 返回值泛型
     * @param time 秒值
     * @return 正确的值：<T> 错误的值：null
     */
    @SuppressWarnings("unchecked")
    public <T> ValueOperations<String, T> setObject(final String key, final T value, final long time) {
        final ValueOperations<String, T> operation = redisTemplate.opsForValue();
        operation.set(key, value, time, TimeUnit.SECONDS);
        return operation;
    }
​
​
    /**
     * 设置对象及失效时间（单位：毫秒）
     *
     * @param key key
     * @param value value值
     * @param <T> 返回值泛型
     * @param time 秒值
     * @return 正确的值：<T> 错误的值：null
     */
    @SuppressWarnings("unchecked")
    public <T> ValueOperations<String, T> setObjectForMillSeconds(final String key, final T value, final long time) {
        final ValueOperations<String, T> operation = redisTemplate.opsForValue();
        operation.set(key, value, time, TimeUnit.MILLISECONDS);
        return operation;
    }
    
    /**
     * 获取对象
     *
     * @param key 键
     * @return 正确的值：Object值对象<br>
     * 错误的值：null
     */
    @SuppressWarnings("unchecked")
    public Object getObject(final String key) {
        final ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
        if (valueOperations == null || !redisTemplate.hasKey(key)) {
            return null;
        }
        final Object object = valueOperations.get(key);
        return object;
    }
    
    /**
     * 从缓存中获取string值
     *
     * @param key
     * @return*/
    @SuppressWarnings("unchecked")
    public String getString(final String key) {
        String value = "";
        final ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
        if (valueOperations != null && redisTemplate.hasKey(key)) {
            final Object object = valueOperations.get(key);
            if (null != object) {
                LOGGER.info("--getString--object not empty");
                value = object.toString();
            } else {
                LOGGER.info("--getString--object empty");
            }
        }
        return value;
    }
```

```text
import com.youjia.orders.redis.RedisManager;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
​
import java.util.Objects;
​
/**
 * @Auther: Yrion
 * @Date: 2019-01-11 23:36
 */
​
public class RedisTest extends OrderProviderApplicationTests {
    
    @Autowired
    private RedisManager redisManager;
    
    @Test
    public void test() {
        controlTime("10000001", 10L);
    }
    
    public void controlTime(String requestId, Long timeOut) {
        
        if (Objects.isNull(requestId) || Objects.isNull(timeOut)) {
            return;
        }
        //something code
        final String value = "value";
        redisManager.setObject(requestId, value, timeOut);
        final long startTime = System.currentTimeMillis();
        System.out.println("开始控制时间");
        //start
        for (; ; ) {
            if (Objects.isNull(redisManager.getObject(requestId))) {
                break;
            }
        }
        final long endTime = System.currentTimeMillis();
        
        final long useTime = endTime - startTime;
        
        System.out.println("一共耗费时间：" + useTime);
    }
}
```

```text
开始控制时间
一共耗费时间：10042
```

本篇博文讲述了在平时工作中，我们可能会遇到的一些关于时间控制的问题，在这个问题上我又进行了进一步的探讨，如何实现优雅的解决问题？我们解决问题不仅仅是要把这个问题解决了，而是要考虑如何更好更秒的解决，这就要善于利用一些中间件或者工具类提供的功能特性，善于发现、及时变通，把这种特性利用到我们的代码中，会对我们的开发起到推波助澜、如虎添翼的作用！

**三：总结**

outPut:

2.2.1:在流程中停留一段时间，通过无限循环来不断的从redis取数值，一旦取到的值为null（redis的键值为null）就退出，这样的写法有点类似于以前CAS的些许味道，通过无限循环比较值。

2.2：在redis中实现时间控制

2.2：redisTemplate模板工具类

2.2: 在springboot中配置redis

**引入**spring-boot-starter-data-redis，这是springboot专门针对redis出的整合依赖库，整合度要比jedis、和redssion都要好，所以推荐这个依赖库：

2.1：maven中引入redis

**二：redis**

3:简单,真正的代码实现起来只有很少，下面会给出代码示范。

2:实现方便灵活，通过key设值可以加入一些唯一性的id来表示业务含义,从而保证业务的稳健实现

1:对代码的侵入性低，不用额外起另外的线程来执行。只需要加入一个方法就可以对单流程的时间控制

通过EXPIRE命令可以设置键的过期时间，一旦超过预设的时间，值就会变成（nil）。利用这一点,加入一些业务参数，我们就可以有效的实现延时的目的。通过redis的过期时间使用redis的好处有以下几点：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/05/640-20200705095343858.png)

在redis中存在一个命令：EXPIRE，这个命令可以设置键存活的时间。一旦超过指定的时间，redis就会将键对应的值给删除掉，因此可以利用这一特性，我们来曲线实现延时功能。在redis的实际命令如下：

1.3:redis延时

这是一个很简单的定时器实现，可以看出它只需要将方法对应的类继承自MyTask就可以实现定时执行，这种方法是可以实现延时的效果，但是它有一个致命的缺点：对代码的侵入性太大，为了实现定时我们不得已将对应的方法封装成一个类，然后放在定时器里执行。这样的、是可以的，但未免也有点太得不偿失了。为此我要更改整个类的结构，对于修改一个东西，我们要尽量按照最简单的方式最好的效果来实现，所以这种方案也应该pass掉。

