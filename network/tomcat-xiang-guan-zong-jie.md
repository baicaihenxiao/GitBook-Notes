# Tomcat相关总结

> [https://elsef.com/2019/11/17/Tomcat%E7%9B%B8%E5%85%B3/](https://elsef.com/2019/11/17/Tomcat%E7%9B%B8%E5%85%B3/)
>
> [https://mp.weixin.qq.com/s?\_\_biz=MzU0MDEwMjgwNA==&mid=2247494377&idx=1&sn=76a635f5a8e6c50038a9749bf687e677&chksm=fb3cf312cc4b7a0401699d92900e74d5fd0facac8999f0983cda73ef87eeefc6f6f2308807b1&mpshare=1&scene=1&srcid=0206FTpE4PEuD7HMjD3DEIQg](https://mp.weixin.qq.com/s?__biz=MzU0MDEwMjgwNA==&mid=2247494377&idx=1&sn=76a635f5a8e6c50038a9749bf687e677&chksm=fb3cf312cc4b7a0401699d92900e74d5fd0facac8999f0983cda73ef87eeefc6f6f2308807b1&mpshare=1&scene=1&srcid=0206FTpE4PEuD7HMjD3DEIQg)



> Tomcat是做Java Web开发时部署服务最受欢迎的容器，关于它的运行机制和调优参数本文进行一定的整理。

#### Architecture

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/02/06/640-141000.jpg)

**配置**

一个经典的配置文件如下所示：

```text
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
      <Listener className="org.apache.catalina.core.AprLifecycleListener" />
      <Listener className="org.apache.catalina.core.JasperListener" />
      <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
      <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
      <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

   <Service name="Catalina">
     <Connector port="8080" protocol="HTTP/1.1"
                connectionTimeout="20000"
                redirectPort="8443" />
     <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
     <Engine name="Catalina" defaultHost="www.testwebapp.com">
       <Host name="www.testwebapp.com"  appBase="webapps"
             unpackWARs="true" autoDeploy="true">
       </Host>
     </Engine>
   </Service>
</Server>
```

我们可以看到它的结构：

* Server组件，顶级元素，是Tomcat的运行实例，一个JVM只包含一个Server组件
* Listener，内置的一些监听器，可以帮我们在发生特定事件的时候起到相应的作用
* Service组件，可以有多个，它是将Connector组件与Container组件包装组合在一起，对外提供服务。该组件中会包含多个Connector组件组件以及一个Container组件。
* Connector组件，隶属于Service组件，可以有多个，如这里分别监听了HTTP/1.1西诶呦和AJP/1.3协议并分别绑定两个独立端口。将Socket请求过来的数据，都封装成Request请求对象，同时将该请求对象传递给容器进行下一步的处理。
* Engine引擎，用来管理多个站点， 一个Service最多只能有一个Engine
* Host，代表一个站点，也可以叫虚拟主机。它指定具体的服务的部署位置，如appBase指定了代码的部署文件夹路径，并且是否支持自动部署和自动解压War包
* Context ：代表一个应用程序，即为我们开发的一个war服务在webapp目录下的各个应用，或者一个WEB-INF 目录以及下面的web.xml 文件；
* Wrapper ：每个Wrapper 封装着一个servlet。

可以从更深层次的视角来查看架构：

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/02/06/640-20210206140959871-141000.jpg)

**HTTP请求处理流程**

当一个HTTP请求到达Tomcat后，所经历的大致流程如下：

* 1、在用户在浏览器中点击页面进行数据请求后，Tomcat本机默认端口8080接收到数据请求，被在那里监听的Coyote HTTP/1.1 Connector获得；
* 2、Connector把封装好的Request请求交由其所在的Service的Engine来处理，并等待Engine的回应；
* 3、Engine获得请求localhost/test/index.jsp，匹配所有的虚拟主机Host；
* 4、Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机），名为localhost的Host获得请求/test/index.jsp，匹配它所拥有的所有的Context。Host匹配到路径为/test的Context（如果匹配不到就把该请求交给路径名为“ ”的Context去处理）；
* 5、path=“/test”的Context获得请求/index.jsp，在它的mapping table中寻找出对应的Servlet。Context匹配到URL PATTERN为\*.jsp的Servlet,对应于JspServlet类；
* 6、构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet（）或doPost（）.执行业务逻辑、数据存储等程序；
* 7、Context把执行完之后的HttpServletResponse对象返回给Host；
* 8、Host把HttpServletResponse响应对象返回至Engine；
* 9、Engine将HttpServletResponse响应对象返回至Connector；
* 10、Connector将HttpServletResponse响应对象返回给客户端的浏览器。

  **Connector**

  Connector是Tomcat的核心组件，从这里 可以看到各种Connector的对比。主要参数如下：

* maxThreads 默认200，表示任意时刻能够并行执行的最大线程数
* minSpareThreads 默认值10，表示任意时刻处于running状态的线程的最小值，包括idle和active两种状态的线程
* maxConnections Tomcat能够接收和处理的并发连接的最大值，超过这个值后的连接将会被放入一个queue中，等待有空闲线程调用。默认值10000（NIO/NIO2）或8192（APR/native）
* acceptCount 默认值100，当没有空闲线程的时候新的TCP请求连接都会放入这个一个queue中，这个是队列的长度值，超过就直接拒绝请求了
* connectionTimeout 单位毫秒，超过这个值Connector将会释放这个idle的连接

Tomcat启动的时候会首先创建minSpareThreads个线程，然后随着负载的增加一直增加到maxThreads，如果此时所有线程都处于busy状态，此后再来的 请求将会被放入queue中（最大容纳acceptCount）直到有空闲线程来执行。当queue满了并且连接数量达到了maxConnections，后续再连接进来的connection将收到 `Connection Refused` 错误。此时应该对线程池的容量做适当调整， 但也不能调整过大，防止服务器负载升高。

**Executor**

它可以更好的在多个Connector之间控制和调度线程池的资源，也便于控制服务器的负载。它的机制和Connector中的默认线程池一样，使用最小和最大线程池参数控制线程个数， 并通过maxQueueSize控制队列的大小，如果超过了能够容纳的容量，则抛出 `RejectedExecutionException` 错误。

**线程模型**

对照下面这张图，可以比较清晰的理解tomcat的Connector和nio模型：

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/02/06/640-20210206140959933-141000.jpg)

* Acceptor：接收socket线程，这里虽然是基于NIO的connector，但是在接收socket方面还是传统的serverSocket.accept\(\)方式， 获得SocketChannel对象，然后封装在一个tomcat的实现类org.apache.tomcat.util.net.NioChannel对象中。然后将NioChannel对象封装在一个PollerEvent对象中，并将PollerEvent对象压入events queue里。这里是个典型的生产者-消费者模式，Acceptor与Poller线程之间通过queue通信，Acceptor是events queue的生产者， Poller是events queue的消费者。
* Poller：Poller线程中维护了一个Selector对象，NIO就是基于Selector来完成逻辑的。在connector中并不止一个Selector， 在socket的读写数据时，为了控制timeout也有一个Selector。可以先把Poller线程中维护的这个Selector标为 主Selector 。Poller是NIO实现的主要线程。首先作为events queue的消费者， 从queue中取出PollerEvent对象，然后将此对象中的channel以 OP\_READ 事件注册到 主Selector 中，然后主Selector执行select操作， 遍历出可以读数据的socket，并从Worker线程池中拿到可用的Worker线程，然后将socket传递给Worker。
* Worker ：Worker线程拿到Poller传过来的socket后，将socket封装在SocketProcessor对象中。然后从Http11ConnectionHandler 中取出Http11NioProcessor对象，从Http11NioProcessor中调用CoyoteAdapter的逻辑。

#### References

* [https://www.datadoghq.com/blog/tomcat-architecture-and-performance/](https://www.datadoghq.com/blog/tomcat-architecture-and-performance/)
* [https://cloud.tencent.com/developer/article/1103077](https://cloud.tencent.com/developer/article/1103077)
* [https://blog.csdn.net/Diamond\_Tao/article/details/87389315](https://blog.csdn.net/Diamond_Tao/article/details/87389315)
* [https://zhuanlan.zhihu.com/p/85448047](https://zhuanlan.zhihu.com/p/85448047)

