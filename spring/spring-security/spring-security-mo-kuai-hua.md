# Spring Security 模块化

[https://mp.weixin.qq.com/s?\_\_biz=MzUzMzQ2MDIyMA==&mid=2247489748&idx=1&sn=2ea12044e4c08fe600dd890e33a8b06a&chksm=faa2fd47cdd574519209192abe452304597d70cc7e46bf00747263f8869cbe14f70bd7cf65e6&mpshare=1&scene=1&srcid=0426VmoQHKtSMAUjLUkxaHrL](https://mp.weixin.qq.com/s?__biz=MzUzMzQ2MDIyMA==&mid=2247489748&idx=1&sn=2ea12044e4c08fe600dd890e33a8b06a&chksm=faa2fd47cdd574519209192abe452304597d70cc7e46bf00747263f8869cbe14f70bd7cf65e6&mpshare=1&scene=1&srcid=0426VmoQHKtSMAUjLUkxaHrL)



最近写了几个Spring Boot组件，项目用什么功能就引入对应的依赖，配置配置就能使用，香的很！那么**Spring Security**能不能也弄成模块化，简单配置一下就可以用上呢？JWT得有，RBAC动态权限更得有！花了小半天就写了个组件，用了一个月感觉还不错。是我一个人爽？还是放出来让大家一起爽？经过我翻来覆去的思想斗争了一个月，最后做出了一个明智的决定，放出来让想直接上手的同学直接使用。源码地址就在下面：

> ❝
>
> [https://gitee.com/felord/security-enhance-spring-boot](https://gitee.com/felord/security-enhance-spring-boot)

## 用法

### 集成

这就是一个Spring Boot Starter，你自己打包、安装。然后引用到项目：

```text
        <dependency>
            <groupId>cn.felord.security</groupId>
            <artifactId>security-enhance-spring-boot-starter</artifactId>
            <version>${version}</version>
        </dependency>
```

另外你需要集成**Spring Cache**，比如Redis Cache:

```text
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
    </dependency>
```

**JWT**会被缓存到以`usrTkn`为key的缓存中，如果你想定制的话，自行实现一个`JwtTokenStorage`并注入**Spring IoC**就可以覆盖下面的配置了：

```text
    @Bean
    @ConditionalOnMissingBean
    public JwtTokenStorage jwtTokenStorage() {
        return new SpringCacheJwtTokenStorage();
    }
```

> ❝
>
> 你应该去了解如何自定义Spring Cache的过期时间。

### 数据库表设计

然后是数据库表设计，这里简单点弄个RBAC的设计，**仅供参考**，你可以根据你们的业务改良。

**用户表**：

| user\_id | username | password |
| :--- | :--- | :--- |
| 1312434534 | felord | {noop}12345 |

**角色表**：

| role\_id | role\_name | role\_code |
| :--- | :--- | :--- |
| 12343667867 | 管理员 | ADMIN |

**用户角色关联表**：

| user\_role\_id | user\_id | role\_id |
| :--- | :--- | :--- |
| 12354657777 | 1312434534 | 12343667867 |

> ❝
>
> 一个用户可以持有多个角色，一个角色在一个用户持有的角色集合中是唯一的。

**资源表**：

| resources\_id | resources\_name | resource\_pattern | method |
| :--- | :--- | :--- | :--- |
| 12543667867 | 根据ID获取商品 | /goods/{goodsId} | GET |

> ❝
>
> 资源其实就是我们写的Spring MVC接口，这里支持ANT风格，但是尽量具体，为了灵活性考虑不推荐使用通配符。

**角色资源表**：

| role\_res\_id | role\_id | resources\_id |
| :--- | :--- | :--- |
| 4545466445 | 12343667867 | 12543667867 |

> ❝
>
> 一个资源可以关联多个角色，一个角色不能重复持有一个资源。

### 实现UserDetailsService

实现用户加载服务接口`UserDetailsService`是Spring Security开发的必要步骤，跟我以前的教程差不多。

```text
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

    UserInfo userInfo = this.lambdaQuery()
            .eq(UserInfo::getUsername, username).one();

    if (Objects.isNull(userInfo)) {
        throw new UsernameNotFoundException("用户：" + username + " 不存在");
    }

    String userId = userInfo.getUserId();
    boolean enabled = userInfo.getEnabled();

    Set<String> roles = iUserRoleService.getRolesByUserId(userId);
    roles.add(“"ANONYMOUS"”);
    Set<GrantedAuthority> roleSet = roles.stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
            .collect(Collectors.toSet());
    return new SecureUser(userId,
            username,
            userInfo.getSecret(),
            enabled,
            enabled,
            enabled,
            enabled,
            roleSet);
}
```

这里要说一下里面为啥要内置一个`ANONYMOUS`角色给用户。如果希望特定的资源对用户全量开放，可配置对应的权限角色编码为`ANONYMOUS`。当某个资源的角色编码为`ANONYMOUS`时，即使不携带**Token**也可以访问。一般情况下匿名能访问的资源不匿名一定能访问，当然你如果不希望这样的规则存在干掉就是了。

### 查询用户的权限集

实现用户角色权限方法`Function<Set<String>, Set<AntPathRequestMatcher>>`并注入**Spring IoC**，根据用户持有的角色集合查询用户可访问的资源列表。这个基于前几天的[动态权限文章](https://mp.weixin.qq.com/s?__biz=MzUzMzQ2MDIyMA==&mid=2247489508&idx=1&sn=f98b9a36ea3614c7be0c7863e4f8897b&scene=21#wechat_redirect)实现的具体可以去了解。也可以根据当前资源的`AntPathRequestMatcher`来查询用户是否持有对应的角色，这个你自行改造。

### 配置

最后就是配置了，跟我以前教程中的配置几乎一样，`application.yaml`的配置为：

```text
# jwt 配置
jwt:
  cert-info:
   # keytool 密钥的 alias 
    alias: felord
    # 密匙密码
    key-password: i6x123akg15v13
    # 路径 这里是在resources 包下
    cert-location: jwt.jks
  claims:
    # jwt iss 字段值
    issuer: https://felord.cn
    # sub 字段
    subject: all
    # 过期秒数
    expires-at: 604800
```

最后别忘记弄个配置类并标记`@EnableSpringSecurity`以启用配置：

```text
@EnableSpringSecurity
@Configuration(proxyBeanMethods = false)
public class SecurityConfiguration {

    /**
     * Function function.
     *
     * @param resourcesService the resources service
     * @return the function
     */
    @Bean
    Function<Set<String>, Set<AntPathRequestMatcher>> function(IResourcesService resourcesService){
        return resourcesService::matchers;
    }

    @Bean
    UserDetailsService userDetailsService(IUserInfoService userInfoService){
        return userInfoService::loadUserByUsername; 
    }

}
```

> ❝
>
> 记得使用`@EnableCaching`开启并配置缓存。

## 使用

登录接口

```text
 POST /login?username=felord&password=12345 HTTP/1.1
 Host: localhost:8080
```

然后会返回一对JWT，返回包含两个token主体

* `accessToken` 用来日常进行请求鉴权，有过期时间。
* `refreshToken` 当`accessToken`过期失效时，用来刷新`accessToken`。

结构为：

```text
{
  "accessToken": {
    "tokenValue": "",
    "issuedAt": {
      "epochSecond": 1616827822,
      "nano": 393000000
    },
    "expiresAt": {
      "epochSecond": 1616831422,
      "nano": 393000000
    },
    "tokenType": {
      "value": "Bearer"
    },
    "scopes": [
      "ROLE_ADMIN",
      "ROLE_ANONYMOUS"
    ]
  },
  "refreshToken": {
    "tokenValue": "",
    "issuedAt": {
      "epochSecond": 1616827822,
      "nano": 393000000
    },
    "expiresAt": null
  },
  "additionalParameters": {}
}
```

调用根据ID获取商品接口时加入Token：

```text
GET /goods/234355451 HTTP/1.1
Host: localhost:8080
Authorization: Bearer eyJraWQImFsZyI6IlJTMjU2In0.eyJzdWIiOiJ1NzgsImlhdCI6MTYxNjkxODk3OCwianRpIjoiNThlOTQktNGVlYzc3MDU0ZDk3In0.ZQcN0FX7_taohqPiC1KnoF7
```

是不是简单多了？觉得好就给个**关注、点赞、转发、再看**。

