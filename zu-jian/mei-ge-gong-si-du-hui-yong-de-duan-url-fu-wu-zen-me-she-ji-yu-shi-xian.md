# 每个公司都会用的短 URL 服务，怎么设计与实现？

[https://mp.weixin.qq.com/s/fEJfB3w3CAoWfLvvvFLDCg](https://mp.weixin.qq.com/s/fEJfB3w3CAoWfLvvvFLDCg)

来源：juejin.im/post/5d10ecab518825795a4d380e

* 前言
* 短URL基础原理
* 服务设计
* 实现

## 前言

想必大家也经常收到垃圾短信吧...短信中的链接一般都是短链接,类似于下图这样:

为什么这里面的url都是短的呢?有什么好处呢?怎么做到的呢?

短url的好处有:

1. 短. 短信和许多平台\(微博\)有字数限制,太长的链接加进去都没有办法写正文了.
2. 好看. 比起一大堆不知所以的参数,短链接更加简洁友好.
3. 方便做一些统计.你点了链接会有人记录然后分析的.
4. 安全. 不暴露访问参数.

这就是为什么我们现在收到的垃圾短信大多数都是短URL的原因了.

那么短URL是怎么做到的呢?

## 短URL基础原理

短URL从生成到使用分为以下几步.

1. 有一个服务,将要发送给你的长URL对应到一个短URL上.例如`www.baidu.com -> www.t.cn/1`
2. 把短url拼接到短信等的内容上发送.
3. 用户点击短URL,浏览器用301/302进行重定向,访问到对应的长URL.
4. 展示对应的内容.

本文主要集中于第一步,即如何将一个长URL对应到短URL上.

## 服务设计

如果你在往长短URL真实的对应关系上想,那么就走远了.

最理想的情况是: 我们用一种算法,对每一个长URL,唯一的转换成短URL.还能保持反向转换的能力.

但是这是不可能的,如果有这样的算法,世界上的所有压缩算法都可以原地去世了.

正确的思路是建立一个发号器,每次有一个新的长URL进来,我们就增加一,并且将新的数值返回.第一个来的url返回"[www.x.cn/0](www.x.cn/0)",第二个返回"[www.x.cn/1](www.x.cn/1)".

接下来以QA形式写几个小问题:

#### 对应关系如何存储?

这个对应数据肯定是要落盘的,不能每次系统重启就重新排号,所以可以采用mysql等数据库来存储.而且如果数据量小且qps低,直接使用数据库的自增主键就可以实现.

#### 如何保证长短链接一一对应?

按照上面的发号器策略,是不能保证长短链接的一一对应的,你连续用同一个URL请求两次,结果值都是不一样的.

为了实现长短链接一一对应,我们需要付出很大的空间代价,尤其是为了快速响应,我们可以需要在内存中做一层缓存,这样子太浪费了.

但是可以实现一些变种的,来实现部分的一一对应, 比如将最近/最热门的对应关系存储在K-V数据库中,这样子可以节省空间的同时,加快响应速度.

#### 短URL的存储

我们返回的短URL一般是将数字转换成32进制,这样子可以更加有效的缩短URL长度,那么32进制的数字对计算机来说只是字符串,怎么存储呢?直接存储字符串对等值查找好找,对范围查找等太不友好了.

其实可以直接存储10进制的数字,这样不仅占用空间少,对查找的支持较好,同时还可以更加方便的转换到更多/更少的进制来进一步缩短URL.

#### 高并发

如果直接存储在MySQL中,当并发请求增大,对数据库的压力太大,可能会造成瓶颈,这时候是可以有一些优化的.

**缓存**

上面_保证长短链接一一对应_中也提到过缓存,这里我们是为了加快程序处理速度.可以将热门的长链接\(需要对长链接进来的次数进行计数\),最近的长链接\(可以使用redis保存最近一个小时的\)等等进行一个缓存,保存在内存中或者类似redis的内存数据库中,如果请求的长URL命中了缓存,那么直接获取对应的短URL进行返回,不需要再进行生成操作.

**批量发号**

每一次发号都需要访问一次MySQL来获取当前的最大号码,并且在获取之后更新最大号码,这个压力是比较大的.

我们可以每次从数据库获取10000个号码,然后在内存中进行发放,当剩余的号码不足1000时,重新向MySQL请求下10000个号码.在上一批号码发放完了之后,批量进行写入.

这样可以将对数据库持续的操作移到代码中进行,并且异步进行获取和写入操作,保证服务的持续高并发.

#### 分布式

上面设计的系统是有单点的,那就是发号器是个单点,容易挂掉.

可以采用分布式服务,分布式的话,如果每一个发号器进行发号之后都需要同步给其他发号器,那未必也太麻烦了.

换一种思路,可以有两个发号器,一个发单号,一个发双号,发号之后不再是递增1,而是递增2.

类比可得,我们可以用1000个服务,分别发放0-999尾号的数字,每次发号之后递增1000.这样做很简单,服务互相之间基本都不用通信,做好自己的事情就好了.

## 实现

由于我懒得写JDBC代码,更懒得弄Mybatis,所以代码中使用到MySQL的地方都使用了Redis.

```java
package util;
​
import redis.clients.jedis.Jedis;
​
/**
 * Created by pfliu on 2019/06/23.
 */
public class ShortUrlUtil {
​
​
    private static final String SHORT_URL_KEY = "SHORT_URL_KEY";
    private static final String LOCALHOST = "http://localhost:4444/";
    private static final String SHORT_LONG_PREFIX = "short_long_prefix_";
    private static final String CACHE_KEY_PREFIX = "cache_key_prefix_";
    private static final int CACHE_SECONDS = 1 * 60 * 60;
​
    private final String redisConfig;
    private final Jedis jedis;
​
    public ShortUrlUtil(String redisConfig) {
        this.redisConfig = redisConfig;
        this.jedis = new Jedis(this.redisConfig);
    }
​
    public String getShortUrl(String longUrl, Decimal decimal) {
        // 查询缓存
        String cache = jedis.get(CACHE_KEY_PREFIX + longUrl);
        if (cache != null) {
            return LOCALHOST + toOtherBaseString(Long.valueOf(cache), decimal.x);
        }
​
        // 自增
        long num = jedis.incr(SHORT_URL_KEY);
        // 在数据库中保存短-长URL的映射关系,可以保存在MySQL中
        jedis.set(SHORT_LONG_PREFIX + num, longUrl);
        // 写入缓存
        jedis.setex(CACHE_KEY_PREFIX + longUrl, CACHE_SECONDS, String.valueOf(num));
        return LOCALHOST + toOtherBaseString(num, decimal.x);
    }
​
    /**
     * 在进制表示中的字符集合
     */
    final static char[] digits = {'0', '1', '2', '3', '4', '5', '6', '7', '8',
            '9', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L',
            'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y',
            'Z', 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z'};
​
    /**
     * 由10进制的数字转换到其他进制
     */
    private String toOtherBaseString(long n, int base) {
        long num = 0;
        if (n < 0) {
            num = ((long) 2 * 0x7fffffff) + n + 2;
        } else {
            num = n;
        }
        char[] buf = new char[32];
        int charPos = 32;
        while ((num / base) > 0) {
            buf[--charPos] = digits[(int) (num % base)];
            num /= base;
        }
        buf[--charPos] = digits[(int) (num % base)];
        return new String(buf, charPos, (32 - charPos));
    }
​
    enum Decimal {
        D32(32),
        D64(64);
​
        int x;
​
        Decimal(int x) {
            this.x = x;
        }
    }
​
​
    public static void main(String[] args) {
​
        for (int i = 0; i < 100; i++) {
            System.out.println(new ShortUrlUtil("localhost").getShortUrl("www.baidudu.com", Decimal.D32));
            System.out.println(new ShortUrlUtil("localhost").getShortUrl("www.baidu.com", Decimal.D64));
        }
    }
}
```

完。

