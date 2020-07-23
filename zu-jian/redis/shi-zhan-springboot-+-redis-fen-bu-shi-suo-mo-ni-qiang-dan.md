# ✔️实战 \| SpringBoot + Redis 分布式锁：模拟抢单

[https://mp.weixin.qq.com/s/U\_JPYoubiAtTWHbISmLULA](https://mp.weixin.qq.com/s/U_JPYoubiAtTWHbISmLULA)



本篇内容主要讲解的是redis分布式锁，这个在各大厂面试几乎都是必备的，下面结合模拟抢单的场景来使用她；本篇不涉及到的redis环境搭建，快速搭建个人测试环境，这里建议使用docker；本篇内容节点如下：

* jedis的nx生成锁
* 如何删除锁
* 模拟抢单动作\(10w个人开抢\)

## jedis的nx生成锁

对于java中想操作redis，好的方式是使用jedis，首先pom中引入依赖：

```text
1         <dependency>
2             <groupId>redis.clients</groupId>
3             <artifactId>jedis</artifactId>
4         </dependency>
```

对于分布式锁的生成通常需要注意如下几个方面：

* 创建锁的策略：redis的普通key一般都允许覆盖，A用户set某个key后，B在set相同的key时同样能成功，如果是锁场景，那就无法知道到底是哪个用户set成功的；这里jedis的setnx方式为我们解决了这个问题，简单原理是：当A用户先set成功了，那B用户set的时候就返回失败，满足了某个时间点只允许一个用户拿到锁。
* 锁过期时间：某个抢购场景时候，如果没有过期的概念，当A用户生成了锁，但是后面的流程被阻塞了一直无法释放锁，那其他用户此时获取锁就会一直失败，无法完成抢购的活动；当然正常情况一般都不会阻塞，A用户流程会正常释放锁；过期时间只是为了更有保障。

下面来上段setnx操作的代码：

\[![&#x590D;&#x5236;&#x4EE3;&#x7801;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/23/copycode-084423.gif)\]\(javascript:void\(0\);\)

```text
 1     public boolean setnx(String key, String val) {
 2         Jedis jedis = null;
 3         try {
 4             jedis = jedisPool.getResource();
 5             if (jedis == null) {
 6                 return false;
 7             }
 8             return jedis.set(key, val, "NX", "PX", 1000 * 60).
 9                     equalsIgnoreCase("ok");
10         } catch (Exception ex) {
11         } finally {
12             if (jedis != null) {
13                 jedis.close();
14             }
15         }
16         return false;
17     }
```

\[![&#x590D;&#x5236;&#x4EE3;&#x7801;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/23/copycode-084423.gif)\]\(javascript:void\(0\);\)

这里注意点在于jedis的set方法，其参数的说明如：

* NX：是否存在key，存在就不set成功
* PX：key过期时间单位设置为毫秒（EX：单位秒）

setnx如果失败直接封装返回false即可，下面我们通过一个get方式的api来调用下这个setnx方法：

```text
1     @GetMapping("/setnx/{key}/{val}")
2     public boolean setnx(@PathVariable String key, @PathVariable String val) {
3         return jedisCom.setnx(key, val);
4     }
```

访问如下测试url，正常来说第一次返回了true，第二次返回了false，由于第二次请求的时候redis的key已存在，所以无法set成功

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/23/348819-20190330153037076-81636031-084425.png)

由上图能够看到只有一次set成功，并key具有一个有效时间，此时已到达了分布式锁的条件。

## 如何删除锁

上面是创建锁，同样的具有有效时间，但是我们不能完全依赖这个有效时间，场景如：有效时间设置1分钟，本身用户A获取锁后，没遇到什么特殊情况正常生成了抢购订单后，此时其他用户应该能正常下单了才对，但是由于有个1分钟后锁才能自动释放，那其他用户在这1分钟无法正常下单（因为锁还是A用户的），因此我们需要A用户操作完后，主动去解锁：

\[![&#x590D;&#x5236;&#x4EE3;&#x7801;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/23/copycode-084423.gif)\]\(javascript:void\(0\);\)

```text
 1     public int delnx(String key, String val) {
 2         Jedis jedis = null;
 3         try {
 4             jedis = jedisPool.getResource();
 5             if (jedis == null) {
 6                 return 0;
 7             }
 8 
 9             //if redis.call('get','orderkey')=='1111' then return redis.call('del','orderkey') else return 0 end
10             StringBuilder sbScript = new StringBuilder();
11             sbScript.append("if redis.call('get','").append(key).append("')").append("=='").append(val).append("'").
12                     append(" then ").
13                     append("    return redis.call('del','").append(key).append("')").
14                     append(" else ").
15                     append("    return 0").
16                     append(" end");
17 
18             return Integer.valueOf(jedis.eval(sbScript.toString()).toString());
19         } catch (Exception ex) {
20         } finally {
21             if (jedis != null) {
22                 jedis.close();
23             }
24         }
25         return 0;
26     }
```

\[![&#x590D;&#x5236;&#x4EE3;&#x7801;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/23/copycode-084423.gif)\]\(javascript:void\(0\);\)

这里也使用了jedis方式，直接执行lua脚本：根据val判断其是否存在，如果存在就del；

其实个人认为通过jedis的get方式获取val后，然后再比较value是否是当前持有锁的用户，如果是那最后再删除，效果其实相当；只不过直接通过eval执行脚本，这样避免多一次操作了redis而已，缩短了原子操作的间隔。\(如有不同见解请留言探讨\)；同样这里创建个get方式的api来测试：

```text
1     @GetMapping("/delnx/{key}/{val}")
2     public int delnx(@PathVariable String key, @PathVariable String val) {
3         return jedisCom.delnx(key, val);
4     }
```

注意的是delnx时，需要传递创建锁时的value，因为通过et的value与delnx的value来判断是否是持有锁的操作请求，只有value一样才允许del；

## 模拟抢单动作\(10w个人开抢\)

有了上面对分布式锁的粗略基础，我们模拟下10w人抢单的场景，其实就是一个并发操作请求而已，由于环境有限，只能如此测试；如下初始化10w个用户，并初始化库存，商品等信息，如下代码：

\[![&#x590D;&#x5236;&#x4EE3;&#x7801;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/23/copycode-084423.gif)\]\(javascript:void\(0\);\)

```text
 1     //总库存
 2     private long nKuCuen = 0;
 3     //商品key名字
 4     private String shangpingKey = "computer_key";
 5     //获取锁的超时时间 秒
 6     private int timeout = 30 * 1000;
 7 
 8     @GetMapping("/qiangdan")
 9     public List<String> qiangdan() {
10 
11         //抢到商品的用户
12         List<String> shopUsers = new ArrayList<>();
13 
14         //构造很多用户
15         List<String> users = new ArrayList<>();
16         IntStream.range(0, 100000).parallel().forEach(b -> {
17             users.add("神牛-" + b);
18         });
19 
20         //初始化库存
21         nKuCuen = 10;
22 
23         //模拟开抢
24         users.parallelStream().forEach(b -> {
25             String shopUser = qiang(b);
26             if (!StringUtils.isEmpty(shopUser)) {
27                 shopUsers.add(shopUser);
28             }
29         });
30 
31         return shopUsers;
32     }
```

\[![&#x590D;&#x5236;&#x4EE3;&#x7801;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/23/copycode-084423.gif)\]\(javascript:void\(0\);\)

有了上面10w个不同用户，我们设定商品只有10个库存，然后通过并行流的方式来模拟抢购，如下抢购的实现：

\[![&#x590D;&#x5236;&#x4EE3;&#x7801;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/23/copycode-084423.gif)\]\(javascript:void\(0\);\)

```text
 1     /**
 2      * 模拟抢单动作
 3      *
 4      * @param b
 5      * @return
 6      */
 7     private String qiang(String b) {
 8         //用户开抢时间
 9         long startTime = System.currentTimeMillis();
10 
11         //未抢到的情况下，30秒内继续获取锁
12         while ((startTime + timeout) >= System.currentTimeMillis()) {
13             //商品是否剩余
14             if (nKuCuen <= 0) {
15                 break;
16             }
17             if (jedisCom.setnx(shangpingKey, b)) {
18                 //用户b拿到锁
19                 logger.info("用户{}拿到锁...", b);
20                 try {
21                     //商品是否剩余
22                     if (nKuCuen <= 0) {
23                         break;
24                     }
25 
26                     //模拟生成订单耗时操作，方便查看：神牛-50 多次获取锁记录
27                     try {
28                         TimeUnit.SECONDS.sleep(1);
29                     } catch (InterruptedException e) {
30                         e.printStackTrace();
31                     }
32 
33                     //抢购成功，商品递减，记录用户
34                     nKuCuen -= 1;
35 
36                     //抢单成功跳出
37                     logger.info("用户{}抢单成功跳出...所剩库存：{}", b, nKuCuen);
38 
39                     return b + "抢单成功，所剩库存：" + nKuCuen;
40                 } finally {
41                     logger.info("用户{}释放锁...", b);
42                     //释放锁
43                     jedisCom.delnx(shangpingKey, b);
44                 }
45             } else {
46                 //用户b没拿到锁，在超时范围内继续请求锁，不需要处理
47 //                if (b.equals("神牛-50") || b.equals("神牛-69")) {
48 //                    logger.info("用户{}等待获取锁...", b);
49 //                }
50             }
51         }
52         return "";
53     }
```

\[![&#x590D;&#x5236;&#x4EE3;&#x7801;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/23/copycode-084423.gif)\]\(javascript:void\(0\);\)

这里实现的逻辑是：

* parallelStream\(\)：并行流模拟多用户抢购
* \(startTime + timeout\) &gt;= System.currentTimeMillis\(\)：判断未抢成功的用户，timeout秒内继续获取锁
* 获取锁前和后都判断库存是否还足够
* jedisCom.setnx\(shangpingKey, b\)：用户获取抢购锁
* 获取锁后并下单成功，最后释放锁：jedisCom.delnx\(shangpingKey, b\)

再来看下记录的日志结果：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/23/348819-20190330153243241-615354177-084428.png)

最终返回抢购成功的用户：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/23/348819-20190330153312380-524759643-084429.png)

