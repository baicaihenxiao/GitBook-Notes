# Spring Boot读取配置属性的常用方式

### [https://mp.weixin.qq.com/s/GeBi7h\_9q7loT256-1ltew](https://mp.weixin.qq.com/s/GeBi7h_9q7loT256-1ltew)

### 1. 前言

在**Spring Boot**项目中我们经常需要读取`application.yml`配置文件的自定义配置，今天就来罗列一下从`yaml`读取配置文件的一些常用手段和方法。

### 2. @Value

首先，会想到使用`@Value`注解，该注解只能去解析`yaml`文件中的简单类型，并绑定到对象属性中去。

```text
felord:
  phone: 182******32
  def:
    name: 码农小胖哥
    blog: felord.cn
    we-chat: MSW_623
  dev:
    name: 码农小胖哥
    blog: felord.cn
    we-chat: MSW_623
  type: JUEJIN
```

对于上面的`yaml`配置，如果我们使用`@Value`注解的话，冒号后面直接有值的`key`才能正确注入对应的值。例如`felord.phone`我们可以通过`@Value`获取，但是`felord.def`不行，因为`felord.def`后面没有直接的值，它还有下一级选项。另外`@Value`不支持`yaml`松散绑定语法，也就是说`felord.def.weChat`获取不到`felord.def.we-chat`的值。

`@Value`是通过使用**Spring**的`SpEL`表达式来获取对应的值的：

```text
// 获取 yaml 中 felord.phone的值 并提供默认值 UNKNOWN
@Value("${felord.phone:UNKNOWN}")
 private String phone;
```

`@Value`的使用场景是只需要获取配置文件中的某项值的情况下，如果我们需要将一个系列的值进行绑定注入就建议使用复杂对象的形式进行注入了。

### 3. @ConfigurationProperties

`@ConfigurationProperties`注解提供了我们将多个配置选项注入复杂对象的能力。它要求我们指定配置的共同前缀。比如我们要绑定`felord.def`下的所有配置项：

```text
package cn.felord.yaml.properties;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

import static cn.felord.yaml.properties.FelordDefProperties.PREFIX;

/**
 * @author felord.cn
 */
@Data
@ConfigurationProperties(PREFIX)
public class FelordDefProperties {
    static final String PREFIX = "felord.def";
    private String name;
    private String blog;
    private String weChat;
}
```

> 我们注意到我们可以使用`weChat`接收`we-chat`的值，因为这种形式支持从驼峰`camel-case`到短横分隔命名`kebab-case`的自动转换。

如果我们使用`@ConfigurationProperties`的话建议配置类命名后缀为`Properties`，比如**Redis**的后缀就是`RedisProperties`,**RabbitMQ**的为`RabbitProperties`。

另外我们如果想进行嵌套的话可以借助于`@NestedConfigurationProperty`注解实现。也可以借助于内部类。这里用内部类实现将开头`yaml`中所有的属性进行注入：

```text
package cn.felord.yaml.properties;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

import static cn.felord.yaml.properties.FelordProperties.PREFIX;


/**
 * 内部类和枚举配置.
 *
 * @author felord.cn
 */
@Data
@ConfigurationProperties(PREFIX)
public class FelordProperties {

    static final String PREFIX = "felord";
    private Def def;
    private Dev dev;
    private Type type;

    @Data
    public static class Def {
        private String name;
        private String blog;
        private String weChat;
    }

    @Data
    public static class Dev {
        private String name;
        private String blog;
        private String weChat;
    }

    public enum Type {
        JUEJIN,
        SF,
        OSC,
        CSDN
    }
}
```

单独使用`@ConfigurationProperties`的话依然无法直接使用配置对象`FelordDefProperties`，因为它并没有被注册为**Spring Bean**。我们可以通过两种方式来使得它生效。

#### 3.1 显式注入 Spring IoC

你可以使用`@Component`、`@Configuration`等注解将`FelordDefProperties`注入**Spring IoC**使之生效。

```text
package cn.felord.yaml.properties;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import static cn.felord.yaml.properties.FelordDefProperties.PREFIX;

/**
 * 显式注入Spring IoC
 * @author felord.cn
 */
@Data
@Component
@ConfigurationProperties(PREFIX)
public class FelordDefProperties {
    static final String PREFIX = "felord.def";
    private String name;
    private String blog;
    private String weChat;
}
```

#### 3.2 @EnableConfigurationProperties

我们还可以使用注解`@EnableConfigurationProperties`进行注册，这样就不需要显式声明配置类为**Spring Bean**了。

```text
package cn.felord.yaml.configuration;

import cn.felord.yaml.properties.FelordDevProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;

/**
 * 使用 {@link EnableConfigurationProperties} 注册 {@link FelordDevProperties}使之生效
 * @author felord.cn
 */
@EnableConfigurationProperties({FelordDevProperties.class})
@Configuration
public class FelordConfiguration {
}
```

该注解需要显式的注册对应的配置类。

#### 3.3 @ConfigurationPropertiesScan

在**Spring Boot 2.2.0.RELEASE**中提供了一个扫描注解`@ConfigurationPropertiesScan`。它可以扫描特定包下所有的被`@ConfigurationProperties`标记的配置类，并将它们进行**IoC**注入。

```text
package cn.felord.yaml;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.ConfigurationPropertiesScan;
import org.springframework.boot.context.properties.EnableConfigurationProperties;

/**
 * {@link ConfigurationPropertiesScan} 同 {@link EnableConfigurationProperties} 二选一
 *
 * @see cn.felord.yaml.configuration.FelordConfiguration
 * @author felord.cn
 */
@ConfigurationPropertiesScan
@SpringBootApplication
public class SpringBootYamlApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootYamlApplication.class, args);
    }

}
```

这非常适合自动注入和批量注入配置类的场景，但是有版本限制，必须在**2.2.0**及以上。

### 4. 总结

日常开发中单个属性推荐使用`@Value`，如果同一组属性为多个则推荐`@ConfigurationProperties`。需要补充一点的是`@ConfigurationProperties`还支持使用 [JSR303](http://mp.weixin.qq.com/s?__biz=MzUzMzQ2MDIyMA==&mid=2247483698&idx=1&sn=9a0e08dc13a828c1b3912247e2ebe07c&chksm=faa2e4a1cdd56db77f7c98bab54155dbec13db72552eb1450a712f886af26889a7ed7cddf426&scene=21#wechat_redirect) 进行属性校验。好了今天的教程就到这里，多多关注：**码农小胖哥** 获取更多的技术干货。相关的**demo** 可通过公众号回复`yaml`获取。

