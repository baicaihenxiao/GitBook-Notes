# 知道Java中的回调机制吗？

{% embed url="https://mp.weixin.qq.com/s/7DPHtmWRfzYMwOaejUyuKA" %}



作者 \| 带妳心菲

来源 \| cnblogs.com/prayjourney/p/9667835.html

### 调用和回调机制

在一个应用系统中, 无论使用何种语言开发, 必然存在模块之间的调用, 调用的方式分为几种:

#### 1.同步调用

![](https://mmbiz.qpic.cn/mmbiz_jpg/eQPyBffYbud3ZAyPnfeWyVoxyUNWJhYAlc5A7eWKQib2vdTHXAnibRdPWk78CjrvXxUeIgDxcOEIYRp1WiaSm5JiaA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

同步调用是最基本并且最简单的一种调用方式, 类A的方法a\(\)调用类B的方法b\(\), 一直等待b\(\)方法执行完毕, a\(\)方法继续往下走. 这种调用方式适用于方法b\(\)执行时间不长的情况, 因为b\(\)方法执行时间一长或者直接阻塞的话, a\(\)方法的余下代码是无法执行下去的, 这样会造成整个流程的阻塞.

#### 2.异步调用

![](https://mmbiz.qpic.cn/mmbiz_jpg/eQPyBffYbud3ZAyPnfeWyVoxyUNWJhYA9CDSEibiaib1hsVP35V5UXiazYGHHyRTR1sJEbNBsoxCmoSlBe8w9gDIZw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

异步调用是为了解决同步调用可能出现阻塞, 导致整个流程卡住而产生的一种调用方式. 类A的方法方法a\(\)通过新起线程的方式调用类B的方法b\(\), 代码接着直接往下执行, 这样无论方法b\(\)执行时间多久, 都不会阻塞住方法a\(\)的执行.

但是这种方式, 由于方法a\(\)不等待方法b\(\)的执行完成, 在方法a\(\)需要方法b\(\)执行结果的情况下（视具体业务而定, 有些业务比如启异步线程发个微信通知、刷新一个缓存这种就没必要）, 必须通过一定的方式对方法b\(\)的执行结果进行监听.

在Java中, 可以使用Future+Callable的方式做到这一点, 具体做法可以参见文章:

> http://www.cnblogs.com/xrq730/p/4872722.html

#### 3.回调

如下图所示, 回调是一种双向的调用方式, 其实而言, 回调也有同步和异步之分, 讲解中是同步回调, 第二个例子使用的是异步回调

![](https://mmbiz.qpic.cn/mmbiz_jpg/eQPyBffYbud3ZAyPnfeWyVoxyUNWJhYAStsT6VNbR5Jcibq5EdzIyJ2MmW6IbdQph4wgV5cB0WWIKLLxDnE52iaw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

回调的思想是:

* 类A的a\(\)方法调用类B的b\(\)方法
* 类B的b\(\)方法执行完毕主动调用类A的callback\(\)方法

通俗而言: 就是A类中调用B类中的某个方法C, 然后B类中反过来调用A类中的方法D, D这个方法就叫回调方法, 这样子说你是不是有点晕晕的, 其实我刚开始也是这样不理解, 看了人家说比较经典的回调方式:

1. class A实现接口CallBack callback——背景1
2. class A中包含一个class B的引用b ——背景2
3. class B有一个参数为callback的方法f\(CallBack callback\) ——背景3
4. A的对象a调用B的方法 f\(CallBack callback\) ——A类调用B类的某个方法 C
5. 然后b就可以在f\(CallBack callback\)方法中调用A的方法 ——B类调用A类的某个方法D

### 回调的种类

回调分为同步回调和异步回调, 假如以买彩票的场景来模拟, 我买彩票, 调用彩票网,给我返回的结果确定是否中奖,同步回调就是,我买了彩票之后, 需要等待彩票网给我返回的结果, 这个时候我不能做其他事情, 我必须等待这个结果, 这就叫同步回调, 同步, 就意味着等待, 我不能去做其他事情, 必须等待。

异步回调就是, 我买了彩票之后, 可以去做其他事情, 然后当彩票网有了结果和消息, 再给我返回消息, 其中最明显的方式就是在得到彩票结果的函数之中, 添加一个其他的方法, 如果我的其他方法可以立即执行, 那么就是异步的\(给出是否中奖需要花费很长的时间\), 而在测试函数之中, 前后两个, 那是发生在测试函数的线程之中的, 肯定是一前一后按照次序的, 在这个地方不是显示同步异步的地点.

#### 同步回调

同步回调和异步回调, 主要体现在其是否需要等待. 同步调用, 如果被调用一方的APi\(第三方API\), 处理问题需要花很长时间, 我们需要等待, 那就是同步回调, 如果调用完之后不需要理解得到结果, 我们调完就走, 去做其他事情, 那就是异步调用, 异步调用需要在我们调用第三方API处, 开启一个新的线程即可, 而同步调用和平常的调用没有任何区别.

#### 例子

OrderResult接口, 其中的方法getOrderResult

```text
public interface OrderResult {
    /**
     * 订购货物的状态
     *
     * @param state
     * @return
     */
    //参数可以不用, 用不用按照自己的实际需求决定
    public String getOrderResult(String state);
}
```

Store类, 商店提供会无预定消息返回的接口, 回调OrderResult接口的方法, 给其返回预订商品的状态, 重点是returnOrderGoodsInfo\(OrderResult order\)方法, 体现了回调的回. Store是被调用的一方, 被调用的一方, 要回过去调用调用一方的方法, 这个方法实际上是回调接口的方法.

```text
public class Store {
    @Getter
    @Setter
    private String name;

    Store(String name) {
        this.name = name;
    }

    /*回调函数, 将结构传给那个我们不能直接调用的方法, 然后获取结果*/
    public String returnOrderGoodsInfo(OrderResult order) {
        String[] s = {"订购中...", "订购失败", "即将发货!", "运输途中...", "已在投递"};
        Random random = new Random();
        int temp = random.nextInt(5);
        String s1 = s[temp];
        return order.getOrderResult(s1);
    }
}
```

SyncBuyer类, 同步顾客类, 其中获取商品的订购状态,orderGoods\(\), 调用了store返回商品调用信息的returnOrderGoodsInfo\(\)方法, 但是在Store类的returnOrderGoodsInfo\(\)方法之中, 以OrderResult接口为参数, 反过来调用了OrderResult接口, 相当于调用了其子类SyncBuyer本身, 以他为参数, 调用了getOrderResult\(String state\)方法, 也就是OrderResult接口的方法, 相当于就完成了一个调用的循环, 然后取到了我们自己无法给出的结果.

这个地方的"循环", 是回调的关键所在, 需要正常调用其他外接提供方法来获取结果的一方, 继承一个回调接口, 实现它, 然后调用第三方的API方法, 第三方在我们调用的方法之中, 以回调结构为参数, 然后调用了接口中的方法, 其中可以返回相应的结果给我们.

需要说明的是, 我们虽然实现了这个接口的方法, 但是我们自己的类之中, 或者说此类本身, 却没法调用这个方法, 也可以说, 此类调用这个方法是不会产生有效的结果的. 回调的回, 就体现在此处, 在Store类之中的returnOrderGoodsInfo\(OrderResult order\)方法之中, 得到了很好的体现.

```text
/*同步, 顾客在商店预订商品, 商店通知顾客预订情况*/
public class SyncBuyer implements OrderResult {
    @Getter
    @Setter
    private Store store;//商店
    @Getter
    @Setter
    private String buyerName;//购物者名
    @Getter
    @Setter
    private String goodsName;//所购商品名

    SyncBuyer(Store store, String buyerName, String goodsName) {
        this.store = store;
        this.buyerName = buyerName;
        this.goodsName = goodsName;
    }

    /*调用从商店返回订购物品的信息*/
    public String orderGoods() {
        String goodsState = store.returnOrderGoodsInfo(this);
        System.out.println(goodsState);
        myFeeling();// 测试同步还是异步, 同步需要等待, 异步无需等待
        return goodsState;

    }

    public void myFeeling() {
        String[] s = {"有点小激动", "很期待!", "希望是个好货!"};
        Random random = new Random();
        int temp = random.nextInt(3);
        System.out.println("我是" + this.getBuyerName() + ", 我现在的感觉: " + s[temp]);
    }

    /*被回调的方法, 我们自己不去调用, 这个方法给出的结果, 是其他接口或者程序给我们的, 我们自己无法产生*/
    @Override
    public String getOrderResult(String state) {
        return "在" + this.getStore().getName() + "商店订购的" + this.getGoodsName() + "玩具, 目前的预订状态是: " + state;
    }
}
```

Test2Callback类, 测试同步回调的结果,

```text
public class Test2Callback {
    public static void main(String[] args) {
        Store wallMart = new Store("沙中路沃尔玛");
        SyncBuyer syncBuyer = new SyncBuyer(wallMart, "小明", "超能铁扇公主");
        System.out.println(syncBuyer.orderGoods());
    }
}
```

#### 异步回调

同步回调和异步回调的代码层面的差别就是是否在我们调用第三方的API处, 为其开辟一条新的线程, 其他并无差异。Java知音公众号内回复”面试题聚合“，送你一份面试题宝典

#### 例子

OrderResult接口, 其中的方法getOrderResult

```text
public interface OrderResult {
    /**
     * 订购货物的状态
     *
     * @param state
     * @return
     */
    //参数可以不用, 用不用按照自己的实际需求决定
    public String getOrderResult(String state);
}
```

Store类, 商店提供会无预定消息返回的接口, 回调OrderResult接口的方法, 给其返回预订商品的状态.

```text
public class Store {
    @Getter
    @Setter
    private String name;

    Store(String name) {
        this.name = name;
    }

    /*回调函数, 将结构传给那个我们不能直接调用的方法, 然后获取结果*/
    public String returnOrderGoodsInfo(OrderResult order) {
        String[] s = {"订购中...", "订购失败", "即将发货!", "运输途中...", "已在投递"};
        Random random = new Random();
        int temp = random.nextInt(5);
        String s1 = s[temp];
        return order.getOrderResult(s1);
    }
}
```

NoSyncBuyer类, 异步调用Store类的returnOrderGoodsInfo\(OrderResult order\)方法, 来返回商品转改的结果.

```text
/*异步*/
@Slf4j
public class NoSyncBuyer implements OrderResult {
    @Getter
    @Setter
    private Store store;//商店
    @Getter
    @Setter
    private String buyerName;//购物者名
    @Getter
    @Setter
    private String goodsName;//所购商品名

    NoSyncBuyer(Store store, String buyerName, String goodsName) {
        this.store = store;
        this.buyerName = buyerName;
        this.goodsName = goodsName;
    }

    /*调用从商店返回订购物品的信息*/
    public String orderGoods() {
        String goodsState = "--";
        MyRunnable mr = new MyRunnable();
        Thread t = new Thread(mr);
        t.start();
        System.out.println(goodsState);
        goodsState = mr.getResult();// 得到返回值
        myFeeling();// 用来测试异步是不是还是按顺序的执行
        return goodsState;
    }

    public void myFeeling() {
        String[] s = {"有点小激动", "很期待!", "希望是个好货!"};
        Random random = new Random();
        int temp = random.nextInt(3);
        System.out.println("我是" + this.getBuyerName() + ", 我现在的感觉: " + s[temp]);
    }

    /*被回调的方法, 我们自己不去调用, 这个方法给出的结果, 是其他接口或者程序给我们的, 我们自己无法产生*/
    @Override
    public String getOrderResult(String state) {
        return "在" + this.getStore().getName() + "商店订购的" + this.getGoodsName() + "玩具, 目前的预订状态是: " + state;
    }

    // 开启另一个线程, 但是没有返回值, 怎么回事
    // 调试的时候, 等待一会儿, 还是可以取到值, 但不是立即取到, 在print显示的时候, 却是null, 需要注意?
    private class MyRunnable implements Runnable {
        @Getter
        @Setter
        private String result;

        @Override
        public void run() {
            try {
                Thread.sleep(10000);
                result = store.returnOrderGoodsInfo(NoSyncBuyer.this);// 匿名函数的时候, 无法return 返回值
            } catch (InterruptedException e) {
                log.error("出大事了, 异步回调有问题了", e);
            }
        }
    }
}
```

Test2Callback类, 测试同步回调和异步回调的结果.

```text
public class Test2Callback {
    public static void main(String[] args) {
        Store wallMart = new Store("沙中路沃尔玛");
        SyncBuyer syncBuyer = new SyncBuyer(wallMart, "小明", "超能铁扇公主");
        System.out.println(syncBuyer.orderGoods());


        System.out.println("\n");
        Store lawson = new Store("沙中路罗森便利店");
        NoSyncBuyer noSyncBuyer = new NoSyncBuyer(lawson, "cherry", "变形金刚");
        System.out.println(noSyncBuyer.orderGoods());
    }
}
```

![](https://mmbiz.qpic.cn/mmbiz_gif/HmHDU48icAtZqViaYZOmSfhUDNRzbGOfTou94iaxgALPicbdZzWOicEQs6KqDrtDSGXfIMib00dYenexpSuE74D6IJkA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

往期推荐[说说Java 中堆和栈的区别吧2020-06-20![](https://mmbiz.qpic.cn/mmbiz_jpg/HmHDU48icAtbIsrDV4CIEk0U4nCzVlhgEvLCs1VlR2ICJ1Pdv0xeicdDw8XlkdoTQlDQP0cia81ZHJyhu7hMwibRfg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](http://mp.weixin.qq.com/s?__biz=MzIxMzQzNzMwMw==&mid=2247484990&idx=1&sn=595abee7e79298846a9fc69e5ce0285a&chksm=97b79826a0c011308b41000bd2a02a6ed7c0f7c5981a09856e641577bad0a97f58b6545da131&scene=21#wechat_redirect)[为什么i++用volatile是存在线程安全问题的？2020-06-19![](https://mmbiz.qpic.cn/mmbiz_jpg/HmHDU48icAtbwEgAbibEia3cyKx8SF0mvMDuk2KR3xg2UnYEbwMzia7ACicCxJa6mEZ6DAXO13rSzqubdXvlAzY8LeQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](http://mp.weixin.qq.com/s?__biz=MzIxMzQzNzMwMw==&mid=2247484970&idx=1&sn=39c712295a97905c8cdd5afd2657e72e&chksm=97b79832a0c011242b049a2d6f50a4a33abf13b046bf4a9f00ab7fa808566434f7850e68e572&scene=21#wechat_redirect)[大厂面试：五轮面试，阿里offer到手！2020-06-18![](https://mmbiz.qpic.cn/mmbiz_jpg/HmHDU48icAtbI0tm7drfcq6M4WGzpO4nay3JoC7yMlOCjdCLvYImRpoiaic2ibmbic4cRu4LjFVzeg61sgQqZLQOP2w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](http://mp.weixin.qq.com/s?__biz=MzIxMzQzNzMwMw==&mid=2247484936&idx=1&sn=f35c41e5483a51785372c75fb9c9b05e&chksm=97b79810a0c0110646b8b8310128e3f6ca53f11de6027dd94211c36e883cbf8ea8efbee1f8f6&scene=21#wechat_redirect)[为什么 https 比 http 更安全？2020-06-17![](https://mmbiz.qpic.cn/mmbiz_jpg/HmHDU48icAtYZjuAyTwJtWfBlcdZHUCudcricmuR607nsNNdqUJuNa08gxum4daCFJ18oK1AljwMJib1Rz4tiaW8ibA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](http://mp.weixin.qq.com/s?__biz=MzIxMzQzNzMwMw==&mid=2247484912&idx=1&sn=095e5380cfafdd885e2a514222476202&chksm=97b79be8a0c012fedc6a72298cccceb132c49d292a1c106fe524bfa9b086af601bcc96beb92c&scene=21#wechat_redirect)[MySQL中，当update修改数据与原数据相同时会再次执行吗？2020-06-16![](https://mmbiz.qpic.cn/mmbiz_jpg/HmHDU48icAtb78BSvL072nAZtxjLIXEHRl3URVcWJianibBYeq03pGqqleicxDOvHsUp4TrCKBNgAr4kZyibxGvGSRw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](http://mp.weixin.qq.com/s?__biz=MzIxMzQzNzMwMw==&mid=2247484838&idx=1&sn=efc1b33b00b65fe60ecfa57e28168144&chksm=97b79bbea0c012a840c91339d6ff705baadf921749304829ad983a608659e8cd49905d084a52&scene=21#wechat_redirect)



