# 面试官：你能告诉我一个请求过来，Spring MVC 是如何找到正确的 Controller 的？

[https://mp.weixin.qq.com/s?\_\_biz=MzUxOTc4NjEyMw==\&mid=2247491254\&idx=2\&sn=f3731a9a3aab7a70009f54283f75c6cf\&chksm=f9f50552ce828c445d582bf2179210408a9ac326ca14f720701b3d281ec800149d78d90512a9\&scene=0\&xtrack=1#rd](https://mp.weixin.qq.com/s?\_\_biz=MzUxOTc4NjEyMw==\&mid=2247491254\&idx=2\&sn=f3731a9a3aab7a70009f54283f75c6cf\&chksm=f9f50552ce828c445d582bf2179210408a9ac326ca14f720701b3d281ec800149d78d90512a9\&scene=0\&xtrack=1#rd)

来源：cnblogs.com/fangjian0423/p/springMVC-request-mapping.html

* 前言
* 源码分析
* 实例
* 资源文件映射
* 总结

## 前言

SpringMVC是目前主流的Web MVC框架之一。

我们使用浏览器通过地址 [http://ip:port/contextPath/path进行访问，SpringMVC是如何得知用户到底是访问哪个Controller中的方法，这期间到底发生了什么。](http://ip/:port/contextPath/path%E8%BF%9B%E8%A1%8C%E8%AE%BF%E9%97%AE%EF%BC%8CSpringMVC%E6%98%AF%E5%A6%82%E4%BD%95%E5%BE%97%E7%9F%A5%E7%94%A8%E6%88%B7%E5%88%B0%E5%BA%95%E6%98%AF%E8%AE%BF%E9%97%AE%E5%93%AA%E4%B8%AAController%E4%B8%AD%E7%9A%84%E6%96%B9%E6%B3%95%EF%BC%8C%E8%BF%99%E6%9C%9F%E9%97%B4%E5%88%B0%E5%BA%95%E5%8F%91%E7%94%9F%E4%BA%86%E4%BB%80%E4%B9%88%E3%80%82)

本文将分析SpringMVC是如何处理请求与Controller之间的映射关系的，让读者知道这个过程中到底发生了什么事情。

## 源码分析

在分析源码之前，我们先了解一下几个东西。

1.这个过程中重要的接口和类。

**HandlerMethod类：**

Spring3.1版本之后引入的。是一个封装了方法参数、方法注解，方法返回值等众多元素的类。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165349914-165350.jpg)img

它的子类InvocableHandlerMethod有两个重要的属性WebDataBinderFactory和HandlerMethodArgumentResolverComposite， 很明显是对请求进行处理的。

InvocableHandlerMethod的子类ServletInvocableHandlerMethod有个重要的属性HandlerMethodReturnValueHandlerComposite，很明显是对响应进行处理的。

ServletInvocableHandlerMethod这个类在HandlerAdapter对每个请求处理过程中，都会实例化一个出来(上面提到的属性由HandlerAdapter进行设置)，分别对请求和返回进行处理。 (RequestMappingHandlerAdapter源码，实例化ServletInvocableHandlerMethod的时候分别set了上面提到的重要属性)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165350213-165350.jpg)img

**MethodParameter类：**

HandlerMethod类中的parameters属性类型，是一个MethodParameter数组。MethodParameter是一个封装了方法参数具体信息的工具类，包括参数的的索引位置，类型，注解，参数名等信息。

HandlerMethod在实例化的时候，构造函数中会初始化这个数组，这时只初始化了部分数据，在HandlerAdapter对请求处理过程中会完善其他属性，之后交予合适的HandlerMethodArgumentResolver接口处理。

以类DeptController为例：

```
@Controller
@RequestMapping(value = "/dept")
public class DeptController {

  @Autowired
  private IDeptService deptService;

  @RequestMapping("/update")
  @ResponseBody
  public String update(Dept dept) {
    deptService.saveOrUpdate(dept);
    return "success";
  }

}
```

(刚初始化时的数据)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165350523-165350.jpg)img

(HandlerAdapter处理后的数据)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165350789-165350.jpg)img

**RequestCondition接口：**

**Spring3.1版本之后引入的。是SpringMVC的映射基础中的请求条件，可以进行combine, compareTo，getMatchingCondition操作。这个接口是映射匹配的关键接口，其中getMatchingCondition方法关乎是否能找到合适的映射。**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165350972-165351.jpg)img

**RequestMappingInfo类：**

Spring3.1版本之后引入的。是一个封装了各种请求映射条件并实现了RequestCondition接口的类。

有各种RequestCondition实现类属性，patternsCondition，methodsCondition，paramsCondition，headersCondition，consumesCondition以及producesCondition，这个请求条件看属性名也了解，分别代表http请求的路径模式、方法、参数、头部等信息。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165351313-165351.jpg)img

**RequestMappingHandlerMapping类：**

处理请求与HandlerMethod映射关系的一个类。

2.Web服务器启动的时候，SpringMVC到底做了什么。

先看AbstractHandlerMethodMapping的initHandlerMethods方法中。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165351569-165351.jpg)img

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165351944-165352.jpg)img

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165352305-165352.jpg)img

我们进入createRequestMappingInfo方法看下是如何构造RequestMappingInfo对象的。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165352542-165352.jpg)img

PatternsRequestCondition构造函数：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165352841-165352.jpg)img

类对应的RequestMappingInfo存在的话，跟方法对应的RequestMappingInfo进行combine操作。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165353124-165353.jpg)img

然后使用符合条件的method来注册各种HandlerMethod。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165353430-165353.jpg)img

下面我们来看下各种RequestCondition接口的实现类的combine操作。

PatternsRequestCondition：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165353666-165353.jpg)img

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165354026-165354.jpg)img

RequestMethodsRequestCondition：

方法的请求条件，用个set直接add即可。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165354235-165354.jpg)img

其他相关的RequestConditon实现类读者可自行查看源码。

最终，RequestMappingHandlerMapping中两个比较重要的属性

private final Map handlerMethods = new LinkedHashMap();

private final MultiValueMap urlMap = new LinkedMultiValueMap();

T为RequestMappingInfo。

构造完成。

我们知道，SpringMVC的分发器DispatcherServlet会根据浏览器的请求地址获得HandlerExecutionChain。

这个过程我们看是如何实现的。

首先看HandlerMethod的获得(直接看关键代码了)：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165354584-165354.jpg)img

这里的比较器是使用RequestMappingInfo的compareTo方法(RequestCondition接口定义的)。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165354913-165355.jpg)img

然后构造HandlerExecutionChain加上拦截器

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165355163-165355.jpg)img

## 实例

写了这么多，来点例子让我们验证一下吧。

```
@Controller
@RequestMapping(value = "/wildcard")
public class TestWildcardController {

  @RequestMapping("/test/**")
  @ResponseBody
  public String test1(ModelAndView view) {
    view.setViewName("/test/test");
    view.addObject("attr", "TestWildcardController -> /test/**");
    return view;
  }

  @RequestMapping("/test/*")
  @ResponseBody
  public String test2(ModelAndView view) {
    view.setViewName("/test/test");
    view.addObject("attr", "TestWildcardController -> /test*");
    return view;
  }

  @RequestMapping("test?")
  @ResponseBody
  public String test3(ModelAndView view) {
    view.setViewName("/test/test");
    view.addObject("attr", "TestWildcardController -> test?");
    return view;
  }

  @RequestMapping("test/*")
  @ResponseBody
  public String test4(ModelAndView view) {
    view.setViewName("/test/test");
    view.addObject("attr", "TestWildcardController -> test/*");
    return view;
  }

}
```

由于这里的每个pattern都带了\*因此，都不会加入到urlMap中，但是handlerMethods还是有的。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165355536-165355.jpg)img

当我们访问：[http://localhost:8888/SpringMVCDemo/wildcard/test1的时候。](http://localhost:8888/SpringMVCDemo/wildcard/test1%E7%9A%84%E6%97%B6%E5%80%99%E3%80%82)

会先根据 "/wildcard/test1" 找urlMap对应的RequestMappingInfo集合，找不到的话取handlerMethods集合中所有的key集合(也就是RequestMappingInfo集合)。

然后进行匹配，匹配根据RequestCondition的getMatchingCondition方法。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165355913-165356.jpg)img

最终匹配到2个RequestMappingInfo：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165356114-165356.jpg)img

然后会使用比较器进行排序。

之前也分析过，比较器是有优先级的。

我们看到，RequestMappingInfo除了pattern，其他属性都是一样的。

我们看下PatternsRequestCondition比较的逻辑：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165356543-165356.jpg)img

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165356872-165356.jpg)img

因此，/test\*的通配符比/test?的多，因此，最终选择了/test?

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165357064-165357.jpg)img

直接比较优先于通配符。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165357241-165357.jpg)img

```
@Controller
@RequestMapping(value = "/priority")
public class TestPriorityController {

  @RequestMapping(method = RequestMethod.GET)
  @ResponseBody
  public String test1(ModelAndView view) {
    view.setViewName("/test/test");
    view.addObject("attr", "其他condition相同，带有method属性的优先级高");
    return view;
  }

  @RequestMapping()
  @ResponseBody
  public String test2(ModelAndView view) {
    view.setViewName("/test/test");
    view.addObject("attr", "其他condition相同，不带method属性的优先级高");
    return view;
  }

}
```

这里例子，其他requestCondition都一样，只有RequestMethodCondition不一样。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165357437-165357.jpg)img

看出，方法多的优先级越多。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165357622-165357.jpg)img

至于其他的RequestCondition，大家自行查看源码吧。

## 资源文件映射

以上分析均是基于Controller方法的映射(RequestMappingHandlerMapping)。

SpringMVC中还有静态文件的映射，SimpleUrlHandlerMapping。

DispatcherServlet找对应的HandlerExecutionChain的时候会遍历属性handlerMappings，这个一个实现了HandlerMapping接口的集合。

由于我们在\*-dispatcher.xml中加入了以下配置：

```
<mvc:resources location="/static/" mapping="/static/**"/>
```

Spring解析配置文件会使用ResourcesBeanDefinitionParser进行解析的时候，会实例化出SimpleUrlHandlerMapping。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165358213-165358.jpg)img

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165358580-165358.jpg)img

其中注册的HandlerMethod为ResourceHttpRequestHandler。

访问地址：[http://localhost:8888/SpringMVCDemo/static/js/jquery-1.11.0.js](http://localhost:8888/SpringMVCDemo/static/js/jquery-1.11.0.js)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165358836-165358.jpg)img

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/03/640-20200803165359067-165359.jpg)img

地址匹配到/static/\*\*。

最终SimpleUrlHandlerMapping找到对应的Handler -> ResourceHttpRequestHandler。

ResourceHttpRequestHandler进行handleRequest的时候，直接输出资源文件的文本内容。

## 总结

大致上整理了一下SpringMVC对请求的处理，包括其中比较关键的类和接口，希望对读者有帮助。

让自己对SpringMVC有了更深入的认识，也为之后分析数据绑定，拦截器、HandlerAdapter等打下基础。
