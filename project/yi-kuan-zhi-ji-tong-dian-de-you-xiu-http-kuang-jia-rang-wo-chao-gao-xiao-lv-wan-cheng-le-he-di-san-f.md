# 一款直击痛点的优秀http框架，让我超高效率完成了和第三方接口的对接

[https://mp.weixin.qq.com/s/jPGHn7hVQ\_Zgb9QpRSZ3Sw](https://mp.weixin.qq.com/s/jPGHn7hVQ_Zgb9QpRSZ3Sw)

来源：22j.co/bUeg

### 1.背景

因为业务关系，要和许多不同第三方公司进行对接。这些服务商都提供基于http的api。但是每家公司提供api具体细节差别很大。有的基于RESTFUL规范，有的基于传统的http规范；有的需要在header里放置签名，有的需要SSL的双向认证，有的只需要SSL的单向认证；有的以JSON 方式进行序列化，有的以XML方式进行序列化。类似于这样细节的差别太多了。

不同的公司API规范不一样，这很正常。但是对于我来说，我如果想要代码变得优雅。我就必须解决一个痛点：

> 不同服务商API那么多的差异点，如何才能维护一套不涉及业务的公共http调用套件。最好通过配置或者简单的参数就能区分开来。进行方便的调用？

我当然知道有很多优秀的大名鼎鼎的http开源框架可以实现任何形式的http调用，在多年的开发经验中我都有使用过。比如apache的httpClient包，非常优秀的Okhttp，jersey client。

这些http开源框架的接口使用相对来说，都不太一样。不管选哪个，在我这个场景里来说，我都不希望在调用每个第三方的http api时写上一堆http调用代码。

所以，在这个场景里，我得对每种不同的http api进行封装。这样的代码才能更加优雅，业务代码和http调用逻辑耦合度更低。

可惜，我比较懒。一来觉得封装起来比较费时间，二来觉对封装这种底层http调用来说，应该有更好的选择。不想自己再去造轮子。

于是，我发现了一款优秀的开源http框架，能屏蔽不同细节http api所带来的所有差异。能通过简单的配置像调用rpc框架一样的去完成极为复杂的http调用。

Forest

> [https://gitee.com/dt\_flys/forest](https://gitee.com/dt_flys/forest)

### 2.上手

Forest支持了Springboot的自动装配，所以只需要引入一个依赖就行

```text
<dependency>
  <groupId>com.dtflys.forest</groupId>
  <artifactId>spring-boot-starter-forest</artifactId>
  <version>1.3.0</version>
</dependency>
```

定义自己的接口类

```text
public interface MyClient {

    @Request(url = "http://baidu.com")
    String simpleRequest();

    @Request(
            url = "http://ditu.amap.com/service/regeo",
            dataType = "json"
    )
    Map getLocation(@DataParam("longitude") String longitude, @DataParam("latitude") String latitude);

}
```

在启动类里配置代理接口类的扫描包

```text
@SpringBootApplication
@ForestScan(basePackages = "com.example.demo.forest")
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

这时候，你就可以从spring容器中注入你的代理接口，像调用本地方法一样去调用http的api了

```text
@Autowired
private MyClient myClient;

@Override
public void yourMethod throws Exception {
    Map result = myClient.getLocation("124.730329","31.463683");
    System.out.println(JSON.toJSONString(result,true));
}
```

日志打印，Forest打印了内部所用的http框架，和实际请求url和返回。当然日志可以通过配置去控制开关。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/26/640-20200726200851370-200851.jpg)

### 3.特点

我觉得对于尤其是做对接第三方api的开发同学来说，这款开源框架能帮你提高很多效率。

Forest 底层封装了2种不同的http框架：Apache httpClient和OKhttp。所以这个开源框架并没有对底层实现进行重复造轮子，而是在易用性上面下足了功夫。

我用Forest最终完成了和多个服务商api对接的项目，这些风格迥异的API，我仅用了1个小时时间就把他们转化为了本地方法。然后项目顺利上线。

Forest作为一款更加高层的http框架，其实你并不需要写很多代码，大多数时候，你仅通过一些配置就能完成http的本地化调用。而这个框架所能覆盖的面，却非常之广，满足你绝大多数的http调用请求。

Forest有以下特点：

* 以Httpclient和OkHttp为后端框架
* 通过调用本地方法的方式去发送Http请求, 实现了业务逻辑与Http协议之间的解耦
* 相比Feign更轻量，不依赖Spring Cloud和任何注册中心
* 支持所有请求方法：GET, HEAD, OPTIONS, TRACE, POST, DELETE, PUT, PATCH
* 支持灵活的模板表达式
* 支持过滤器来过滤传入的数据
* 基于注解、配置化的方式定义Http请求
* 支持Spring和Springboot集成
* 实现JSON和XML的序列化和反序列化
* 支持JSON转换框架: Fastjson,Jackson, Gson
* 支持JAXB形式的XML转换
* 支持SSL的单向和双向加密
* 支持http连接池的设定
* 可以通过OnSuccess和OnError接口参数实现请求结果的回调
* 配置简单，一般只需要@Request一个注解就能完成绝大多数请求的定义
* 支持异步请求调用

面试题推荐：[100期面试题汇总](http://mp.weixin.qq.com/s?__biz=MzIyNDU2ODA4OQ==&mid=2247484532&idx=1&sn=1c243934507d79db4f76de8ed0e5727f&chksm=e80db202df7a3b14fe7077b0fe5ec4de4088ce96a2cde16cbac21214956bd6f2e8f51193ee2b&scene=21#wechat_redirect)

### 4.两个很棒的功能

这里不对使用方式和配置方式一一描述，有兴趣的可以去阅读详细文档：

> [https://dt\_flys.gitee.io/forest](https://dt_flys.gitee.io/forest)

这里只想分析这个框架2个我认为比较好的功能

#### 4.1 模板表达式和参数的映射绑定功能

模板表达式在使用的时候特别方便，举个栗子

```text
@Request(
    url = "${0}/send?un=${1}&pw=${2}&ph=${3}&ct=${4}",
    type = "get",
    dataType = "json"
)
public Map send(
    String base,
    String userName,
    String password,
    String phone,
    String content
);
```

上述是用序号下标进行取值，也可以通过名字进行取值：

```text
@Request(
    url = "${base}/send?un=${un}&pw=${pw}&ph=${3}&ct=${ct}",
    type = "get",
    dataType = "json"
)
public Map send(
    @DataVariable("base") String base,
    @DataVariable("un") String userName,
    @DataVariable("pw") String password,
    @DataVariable("ph") String phone,
    @DataVariable("ct") String content
);
```

甚至于可以这样简化写：

```text
@Request(
    url = "${base}/send",
    type = "get",
    dataType = "json"
)
public Map send(
    @DataVariable("base") String base,
    @DataParam("un") String userName,
    @DataParam("pw") String password,
    @DataParam("ph") String phone,
    @DataParam("ct") String content
);
```

以上三种写法是等价的

当然你也可以把参数绑定到header和body里去，你甚至于可以用一些表达式简单的把对象序列化成json或者xml：

```text
@Request(
    url = "${base}/pay",
   contentType = "application/json",
    type = "post",
    dataType = "json",
    headers = {"Authorization: ${1}"},
    data = "${json($0)}"
)
public PayResponse pay(PayRequest request, String auth);
```

当然数据绑定这块详情请参阅文档

#### 4.2 对HTTPS的支持

以前用其他http框架处理https的时候，总觉得特别麻烦，尤其是双向证书。每次碰到问题也只能去baidu。然后根据别人的经验来修改自己的代码。

Forest对于这方面也想的很周到，底层完美封装了对https单双向证书的支持。也是只要通过简单的配置就能迅速完成。举个双向证书栗子：

```text
@Request(
    url = "${base}/pay",
   contentType = "application/json",
    type = "post",
    dataType = "json",
   keyStore = "pay-keystore",
   data = "${json($0)}"
)
public PayResponse pay(PayRequest request);
```

其中pay-keystore对应着application.yml里的ssl-key-stores

```text
forest:
  ...
  ssl-key-stores:
    - id: pay-keystore
      file: test.keystore
      keystore-pass: 123456
      cert-pass: 123456
      protocols: SSLv3
```

这样设置，就ok了，剩下的，就是本地代码形式的调用了。

### 5.最后

Forest有很多其他的功能设定，如果感兴趣的同学还请仔细去阅读文档和示例。

但是我想说的是，相信看到这里，很多人一定会说，这不就是Feign吗？

我在开发Spring Cloud项目的时候，也用过一段时间Feign，个人感觉Forest的确在配置和用法上和Feign的设计很像，但Feign的角色更多是作为Spring Cloud生态里的一个成员。充当RPC通信的角色，其承担的不仅是http通讯，还要对注册中心下发的调用地址进行负载均衡。

而Forest这个开源项目其定位则是一个高阶的http工具，主打友好和易用性。从使用角度出发，个人感觉Forest配置性更加简单直接。提供的很多功能也能解决很多人的痛点。

开源精神难能可贵，好的开源需要大家的添砖加瓦和支持。希望这篇文章能给大家在选择http客户端框架时带来一个新的选择：Forest

