# 基于SLF4J MDC机制实现日志的链路追踪

[https://my.oschina.net/xiaolyuh/blog/1823392](https://my.oschina.net/xiaolyuh/blog/1823392)

[https://mp.weixin.qq.com/s/0pfm4\_hpOats1CqdXDFJdg](https://mp.weixin.qq.com/s/0pfm4_hpOats1CqdXDFJdg)

## 问题描述

最近经常做线上问题的排查，而排查问题用得最多的方式是查看日志，但是在现有系统中，各种无关日志穿行其中，导致我没办法快速的找出用户在一次请求中所有的日志。

## 问题分析

我们没办法快速定位用户在一次请求中对应的所有日志，或者说是定位某个用户操作的所有日志，那是因为我们在输出的日志的时候没把请求的唯一标示或者说是用户身份标示输出到我们的日志中，导致我们没办法根据一个请求或者用户身份标示来做日志的过滤。所以我们在记录日志的是后把请求的唯一标示（sessionId）或者身份标示（userId） 记录到日志中这个问题就可以得到很好的解决了。

## 解决方案

1. 在每次请求的时候，获取到请求的sessionId（或者自己生成一个伪sessionId），并在每次输出log的时候将这个sessionId输出到日志中。这个方式实现简单，代码侵入型强，每次输出都会多输出一个sessionId参数，工作量大，但是可控粒度高。
2. 我们使用Logback的MDC机制，日志模板中加入sessionId格式。在日志输出格式中指定输出sessionId。如：

```text
%d{yyyy-MM-dd HH:mm:ss.SSS}  [%X{sessionId}] -%5p ${PID:-} [%15.15t] %-40.40logger{39} : %m%n
```

这种方式工作量小，代码侵入小，易扩展，但是可控粒度低。

## 方案说明

第一种方案很简单，也很容易实现，就是在输出日志的时候多输出一个参数，如：

```text
logger.info("sessionId: {}, message: {}", sessionId, "日志信息");
```

我们这里主要说一下第二种方式的实现。

> 实现思路，这里以Spring MVC为例：
>
> 1. 新建一个日志拦截器，在拦截所有请求，在处理请求前将sessionId放到MDC中，在处理完请求后清除MDC的内容。这里就解决了80%的问题。
> 2. 在原来版本中新起线程时MDC会自动将父线程的MDC内容复制给子线程，因为MDC内部使用的是InheritableThreadLocal，但是因为性能问题在最新的版本中被取消了，所以子线程不会自动获取到父线程的MDC内容。官方建议我们在父线程新建子线程之前调用MDC.getCopyOfContextMap\(\)方法将MDC内容取出来传给子线程，子线程在执行操作前先调用MDC.setContextMap\(\)方法将父线程的MDC内容设置到子线程。
> 3. 设置日志输出格式
>
> ```text
> %d{yyyy-MM-dd HH:mm:ss.SSS}  [%X{sessionId}] -%5p ${PID:-} [%15.15t] %-40.40logger{39} : %m%n
> ```

### 拦截器 LogInterceptor

```text
package com.xiaolyuh.interceptors;
​
import org.slf4j.MDC;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
​
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.UUID;
​
/**
 * 日志拦截器组件，在输出日志中加上sessionId
 *
 * @author yuhao.wang3
 */
public class LogInterceptor extends HandlerInterceptorAdapter {
    /**
     * 会话ID
     */
    private final static String SESSION_KEY = "sessionId";
​
    @Override
    public void afterCompletion(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, Exception arg3)
            throws Exception {
​
        // 删除SessionId
        MDC.remove(SESSION_KEY);
    }
​
    @Override
    public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1,
                           Object arg2, ModelAndView arg3) throws Exception {
    }
​
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response, Object handler) throws Exception {
​
        // 设置SessionId
        String token = UUID.randomUUID().toString().replace("-", "");
        MDC.put(SESSION_KEY, token);
        return true;
    }
}
```

### 注册拦截器

```text
/**
 * WEB MVC配置类
 *
 * @author yuhao.wang3
 */
@Configuration
public class WebMvcConfigurer extends WebMvcConfigurerAdapter {
​
    /**
     * 把我们的拦截器注入为bean
     *
     * @return
     */
    @Bean
    public HandlerInterceptor logInterceptor() {
        return new LogInterceptor();
    }
​
    /**
     * 注册拦截器
     *
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // addPathPatterns 用于添加拦截规则, 这里假设拦截 /url 后面的全部链接
        // excludePathPatterns 用户排除拦截
        registry.addInterceptor(logInterceptor()).addPathPatterns("/**");
        super.addInterceptors(registry);
    }
}
```

### 扩展ThreadPoolTaskExecutor线程池

扩展ThreadPoolTaskExecutor线程池的主要目的是实现将父线程的MDC内容复制给子线程。

```text
package com.xiaolyuh.utils;
​
import org.slf4j.MDC;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
​
import java.util.Map;
​
/**
 * 这是{@link ThreadPoolTaskExecutor}的一个简单替换，可以在每个任务之前设置子线程的MDC数据。
 * <p/>
 * 在记录日志的时候，一般情况下我们会使用MDC来存储每个线程的特有参数，如身份信息等，以便更好的查询日志。
 * 但是Logback在最新的版本中因为性能问题，不会自动的将MDC的内存传给子线程。所以Logback建议在执行异步线程前
 * 先通过MDC.getCopyOfContextMap()方法将MDC内存获取出来，再传给线程。
 * 并在子线程的执行的最开始调用MDC.setContextMap(context)方法将父线程的MDC内容传给子线程。
 * <p>
 * https://logback.qos.ch/manual/mdc.html
 *
 * @author yuhao.wang3
 */
public class MdcThreadPoolTaskExecutor extends ThreadPoolTaskExecutor {
​
    /**
     * 所有线程都会委托给这个execute方法，在这个方法中我们把父线程的MDC内容赋值给子线程
     * https://logback.qos.ch/manual/mdc.html#managedThreads
     *
     * @param runnable
     */
    @Override
    public void execute(Runnable runnable) {
        // 获取父线程MDC中的内容，必须在run方法之前，否则等异步线程执行的时候有可能MDC里面的值已经被清空了，这个时候就会返回null
        Map<String, String> context = MDC.getCopyOfContextMap();
        super.execute(() -> run(runnable, context));
    }
​
    /**
     * 子线程委托的执行方法
     *
     * @param runnable {@link Runnable}
     * @param context  父线程MDC内容
     */
    private void run(Runnable runnable, Map<String, String> context) {
        // 将父线程的MDC内容传给子线程
        MDC.setContextMap(context);
        try {
            // 执行异步操作
            runnable.run();
        } finally {
            // 清空MDC内容
            MDC.clear();
        }
    }
}
```

### 扩展Hystrix

#### 扩展Hystrix线程池隔离支持日志链路跟踪

```text
/**
 * Hystrix线程池隔离支持日志链路跟踪
 *
 * @author yuhao.wang3
 */
public class MdcHystrixConcurrencyStrategy extends HystrixConcurrencyStrategy {
​
    @Override
    public <T> Callable<T> wrapCallable(Callable<T> callable) {
        return new MdcAwareCallable(callable, MDC.getCopyOfContextMap());
    }
​
    private class MdcAwareCallable<T> implements Callable<T> {
​
        private final Callable<T> delegate;
​
        private final Map<String, String> contextMap;
​
        public MdcAwareCallable(Callable<T> callable, Map<String, String> contextMap) {
            this.delegate = callable;
            this.contextMap = contextMap != null ? contextMap : new HashMap();
        }
​
        @Override
        public T call() throws Exception {
            try {
                MDC.setContextMap(contextMap);
                return delegate.call();
            } finally {
                MDC.clear();
            }
        }
    }
}
```

#### 配置Hystrix

```text
@Configuration
public class HystrixConfig {
​
    //用来拦截处理HystrixCommand注解
    @Bean
    public HystrixCommandAspect hystrixAspect() {
        return new HystrixCommandAspect();
    }
​
    @PostConstruct
    public void init() {
        HystrixPlugins.getInstance().registerConcurrencyStrategy(new MdcHystrixConcurrencyStrategy());
    }
​
}
```

### Logback配置

```text
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} [%X{sessionId}] %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    <property name="FILE_LOG_PATTERN"
              value="${FILE_LOG_PATTERN:-%d{yyyy-MM-dd HH:mm:ss.SSS} [%X{sessionId}] ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
​
    <appender name="ASYNC_FILE" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>2048</queueSize>
        <includeCallerData>true</includeCallerData>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref="FILE"/>
    </appender>
​
    <logger name="com.xiaolyuh" level="debug" additivity="true"/>
</configuration>  
```

### 测试类

```text
@Service
public class LogService {
    Logger logger = LoggerFactory.getLogger(LogService.class);
​
    public void log() {
        logger.debug("==============================================");
        ThreadTaskUtils.run(() -> run());
        FutureTask<String> futureTask = new FutureTask<String>(() -> call());
        ThreadTaskUtils.run(futureTask);
        try {
            logger.debug("===================: {}", futureTask.get());
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
        logger.debug("==============================================");
    }
​
    private String call() {
        logger.debug("11111111111");
        return "3333";
    }
​
    public void run() {
        logger.debug("222222222222222");
    }
}
```

\#\# 日志输出示例

```text
2018-06-02 23:11:02.376 [a114db8f4be942d891407d57ff74276d] DEBUG 12828 --- [http-nio-80-exec-9] com.xiaolyuh.service.LogService          : ==============================================
2018-06-02 23:11:02.376 [a114db8f4be942d891407d57ff74276d] DEBUG 12828 --- [MdcThreadPoolTaskExecutor-4] com.xiaolyuh.service.LogService          : 222222222222222
2018-06-02 23:11:02.377 [a114db8f4be942d891407d57ff74276d] DEBUG 12828 --- [MdcThreadPoolTaskExecutor-1] com.xiaolyuh.service.LogService          : 11111111111
2018-06-02 23:11:02.377 [a114db8f4be942d891407d57ff74276d] DEBUG 12828 --- [http-nio-80-exec-9] com.xiaolyuh.service.LogService          : ===================: 3333
2018-06-02 23:11:02.377 [a114db8f4be942d891407d57ff74276d] DEBUG 12828 --- [http-nio-80-exec-9] com.xiaolyuh.service.LogService          : ==============================================
2018-06-02 23:11:02.536 [8657ed4f5267489aa323f9422974002b] DEBUG 12828 --- [http-nio-80-exec-2] com.xiaolyuh.service.LogService          : ==============================================
2018-06-02 23:11:02.536 [8657ed4f5267489aa323f9422974002b] DEBUG 12828 --- [MdcThreadPoolTaskExecutor-5] com.xiaolyuh.service.LogService          : 222222222222222
2018-06-02 23:11:02.536 [8657ed4f5267489aa323f9422974002b] DEBUG 12828 --- [MdcThreadPoolTaskExecutor-3] com.xiaolyuh.service.LogService          : 11111111111
2018-06-02 23:11:02.536 [8657ed4f5267489aa323f9422974002b] DEBUG 12828 --- [http-nio-80-exec-2] com.xiaolyuh.service.LogService          : ===================: 3333
2018-06-02 23:11:02.536 [8657ed4f5267489aa323f9422974002b] DEBUG 12828 --- [http-nio-80-exec-2] com.xiaolyuh.service.LogService          : ==============================================
2018-06-02 23:11:02.728 [e85380fb1554463ca156318b0a3ff7c2] DEBUG 12828 --- [http-nio-80-exec-3] com.xiaolyuh.service.LogService          : ==============================================
2018-06-02 23:11:02.728 [e85380fb1554463ca156318b0a3ff7c2] DEBUG 12828 --- [MdcThreadPoolTaskExecutor-2] com.xiaolyuh.service.LogService          : 222222222222222
2018-06-02 23:11:02.729 [e85380fb1554463ca156318b0a3ff7c2] DEBUG 12828 --- [MdcThreadPoolTaskExecutor-4] com.xiaolyuh.service.LogService          : 11111111111
2018-06-02 23:11:02.729 [e85380fb1554463ca156318b0a3ff7c2] DEBUG 12828 --- [http-nio-80-exec-3] com.xiaolyuh.service.LogService          : ===================: 3333
2018-06-02 23:11:02.729 [e85380fb1554463ca156318b0a3ff7c2] DEBUG 12828 --- [http-nio-80-exec-3] com.xiaolyuh.service.LogService          : ==============================================
```

## 源码

[https://github.com/wyh-spring-ecosystem-student/spring-boot-student/tree/releases](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fwyh-spring-ecosystem-student%2Fspring-boot-student%2Ftree%2Freleases)

spring-boot-student-log 工程

