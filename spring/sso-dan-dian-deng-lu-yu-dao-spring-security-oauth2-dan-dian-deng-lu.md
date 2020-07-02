# SSO 单点登录 芋道 Spring Security OAuth2 单点登录

{% embed url="https://mp.weixin.qq.com/s/Fz9uAfxSH\_4W98gMBQGczQ" %}

{% embed url="http://www.iocoder.cn/Spring-Security/OAuth2-learning-sso/" %}





## 1. 概述

在前面的文章中，我们学习了 Spring Security OAuth 的**简单**使用。

* [《芋道 Spring Security OAuth2 入门》](http://www.iocoder.cn/Spring-Security/OAuth2-learning/?self)
* [《芋道 Spring Security OAuth2 存储器》](http://www.iocoder.cn/Spring-Security/OAuth2-learning-store/?self)

今天我们来搞**波“大”**的，通过 Spring Security OAuth 实现一个**单点登录**的功能。

可能会有**女**粉丝不太了解单点登录是什么？单点登录，英文是 **Single Sign On**，简称为 **SSO**，指的是当有**多个**系统需要登录时，用户只需要登录一个**统一**的登录系统，而无需在**多个**系统重复登录。

举个最常见的**例子**，我们在浏览器中使用阿里“全家桶”：

> 求助信：麻烦有认识阿里的胖友，让他们给打下钱。。。

* 淘宝：[https://www.taobao.com](https://www.taobao.com/)
* 天猫：[https://www.tmall.com](https://www.tmall.com/)
* 飞猪：[https://www.fliggy.com](https://www.fliggy.com/)
* …

我们只需要在**统一登录系统**（[https://login.taobao.com](https://login.taobao.com/)）进行登录即可，而后就可以“愉快”的自由剁手，并且无需分别在淘宝、天猫、飞猪等等系统重新登录。

![&#x767B;&#x5F55;&#x7CFB;&#x7EDF;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/01.png)

> 友情提示：更多单点登录的介绍，可见[《维基百科 —— 单点登录》](https://zh.wikipedia.org/wiki/單一登入)。

下面，我们正式搭建 Spring Security OAuth 实现 SSO 的**示例项目**，如下图所示：

![&#x9879;&#x76EE;&#x7ED3;&#x6784;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/02.png)

* 创建 [`lab-68-demo21-authorization-server-on-sso`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/) 项目，作为**统一登录系统**。

  > 旁白君：机智的胖友，是不是发现这个项目和**授权**服务器非常相似！！！

* 创建 [`lab-68-demo21-resource-server-on-sso`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-resource-server-on-sso/) 项目，模拟需要登录的 **XXX 系统**。

  > 旁白君：机智的胖友，是不是发现这个项目和**资源**服务器非常相似！！！

## 2. 搭建统一登录系统

> 示例代码对应仓库：
>
> * 统一登录系统：[`lab-68-demo21-authorization-server-on-sso`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/)

创建 [`lab-68-demo21-authorization-server-on-sso`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/) 项目，作为**统一登录系统**。

> 友情提示：整个实现代码，和我们前文看到的**授权**服务器是基本一致的。

### 2.1 初始化数据库

在 [`resources/db`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/src/main/resources/db/) 目录下，有四个 SQL 脚本，分别用于初始化 User 和 OAuth 相关的表。

![SQL &#x811A;&#x672C;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/11.png)

#### 2.1.1 初始化 OAuth 表

① 执行 [`oauth_schema.sql`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/src/main/resources/db/oauth_schema.sql) 脚本，创建数据库**表结构**。

```text
drop table if exists oauth_client_details;
create table oauth_client_details (
  client_id VARCHAR(255) PRIMARY KEY,
  resource_ids VARCHAR(255),
  client_secret VARCHAR(255),
  scope VARCHAR(255),
  authorized_grant_types VARCHAR(255),
  web_server_redirect_uri VARCHAR(255),
  authorities VARCHAR(255),
  access_token_validity INTEGER,
  refresh_token_validity INTEGER,
  additional_information VARCHAR(4096),
  autoapprove VARCHAR(255)
);

create table if not exists oauth_client_token (
  token_id VARCHAR(255),
  token LONG VARBINARY,
  authentication_id VARCHAR(255) PRIMARY KEY,
  user_name VARCHAR(255),
  client_id VARCHAR(255)
);

create table if not exists oauth_access_token (
  token_id VARCHAR(255),
  token LONG VARBINARY,
  authentication_id VARCHAR(255) PRIMARY KEY,
  user_name VARCHAR(255),
  client_id VARCHAR(255),
  authentication LONG VARBINARY,
  refresh_token VARCHAR(255)
);

create table if not exists oauth_refresh_token (
  token_id VARCHAR(255),
  token LONG VARBINARY,
  authentication LONG VARBINARY
);

create table if not exists oauth_code (
  code VARCHAR(255), authentication LONG VARBINARY
);

create table if not exists oauth_approvals (
    userId VARCHAR(255),
    clientId VARCHAR(255),
    scope VARCHAR(255),
    status VARCHAR(10),
    expiresAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    lastModifiedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

结果如下图所示：

![&#x8868;&#x7ED3;&#x6784;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/12.png)

| 表 | 作用 |
| :--- | :--- |
| `oauth_access_token` | OAuth 2.0 **访问**令牌 |
| `oauth_refresh_token` | OAuth 2.0 **刷新**令牌 |
| `oauth_code` | OAuth 2.0 **授权码** |
| `oauth_client_details` | OAuth 2.0 **客户端** |
| `oauth_client_token` |  |
| `oauth_approvals` |  |

> 旁白君：这里的表结构设计，我们可以借鉴参考，实现自己的 OAuth 2.0 的功能。

② 执行 [`oauth_data.sql`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/src/main/resources/db/oauth_data.sql) 脚本，插入一个客户端记录。

```text
INSERT INTO oauth_client_details
    (client_id, client_secret, scope, authorized_grant_types,
    web_server_redirect_uri, authorities, access_token_validity,
    refresh_token_validity, additional_information, autoapprove)
VALUES
    ('clientapp', '112233', 'read_userinfo,read_contacts',
    'password,authorization_code,refresh_token', 'http://127.0.0.1:9090/login', null, 3600, 864000, null, true);
```

> **注意**！这条记录的 `web_server_redirect_uri` 字段，我们设置为 [http://127.0.0.1:9090/login，这是稍后我们搭建的](http://127.0.0.1:9090/login，这是稍后我们搭建的) XXX 系统的回调地址。
>
> * 统一登录系统采用 OAuth 2.0 的**授权码**模式进行授权。
> * 授权成功后，浏览器会跳转 [http://127.0.0.1:9090/login](http://127.0.0.1:9090/login) 回调地址，然后 XXX 系统会通过**授权码**向统一登录系统获取**访问令牌**。
>
> 通过这样的方式，完成一次**单点登录**的过程。

结果如下图所示：

![\`oauth\_client\_details\` &#x8868;&#x8BB0;&#x5F55;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/12-20200702220548407.png)

#### 2.1.2 初始化 User 表

① 执行 [`user_schema.sql`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/src/main/resources/db/user_data.sql) 脚本，创建数据库**表结构**。

```text
DROP TABLE IF EXISTS `authorities`;
CREATE TABLE `authorities` (
  `username` varchar(50) NOT NULL,
  `authority` varchar(50) NOT NULL,
  UNIQUE KEY `ix_auth_username` (`username`,`authority`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `username` varchar(50) NOT NULL,
  `password` varchar(500) NOT NULL,
  `enabled` tinyint(1) NOT NULL,
  PRIMARY KEY (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

结果如下图所示：

![&#x8868;&#x7ED3;&#x6784;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/13.png)

| 表 | 作用 |
| :--- | :--- |
| `users` | **用户**表 |
| `authorities` | **授权**表，例如用户拥有的角色 |

② 执行 [`user_data.sql`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/src/main/resources/db/user_data.sql) 脚本，插入一个用户记录和一个授权记录。

```text
INSERT INTO `authorities` VALUES ('yunai', 'ROLE_USER');

INSERT INTO `users` VALUES ('yunai', '112233', '1');
```

结果如下图所示：

![\`users\` &#x548C; \`authorities\` &#x8868;&#x8BB0;&#x5F55;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/14.png)

### 2.2 引入依赖

创建 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/pom.xml) 文件，引入 Spring Security OAuth 依赖。

```text
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>lab-68</artifactId>
        <groupId>cn.iocoder.springboot.labs</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-68-demo21-authorization-server-on-sso</artifactId>

    <properties>
        <!-- 依赖相关配置 -->
        <spring.boot.version>2.2.4.RELEASE</spring.boot.version>
        <!-- 插件相关配置 -->
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.source>1.8</maven.compiler.source>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- 实现对 Spring MVC 的自动配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 实现对 Spring Security OAuth2 的自动配置 -->
        <dependency>
            <groupId>org.springframework.security.oauth.boot</groupId>
            <artifactId>spring-security-oauth2-autoconfigure</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>

        <!-- 实现对数据库连接池的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency> <!-- 本示例，我们使用 MySQL -->
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.48</version>
        </dependency>

    </dependencies>

</project>
```

### 2.3 配置文件

创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/src/main/resources/application.yaml) 配置文件，添加**数据库连接池**的配置：

```text
spring:
  # datasource 数据源配置内容，对应 DataSourceProperties 配置属性类
  datasource:
    url: jdbc:mysql://127.0.0.1:43063/demo-68-authorization-server-sso?useSSL=false&useUnicode=true&characterEncoding=UTF-8
    driver-class-name: com.mysql.jdbc.Driver
    username: root # 数据库账号
    password: 123456 # 数据库密码
```

### 2.4 SecurityConfig

创建 [SecurityConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/src/main/java/cn/iocoder/springboot/lab68/authorizationserverdemo/config/SecurityConfig.java) 配置类，通过 **Spring Security** 提供**用户认证**的功能。代码如下：

```text
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 数据源 DataSource
     */
    @Autowired
    private DataSource dataSource;

    @Override
    @Bean(name = BeanIds.AUTHENTICATION_MANAGER)
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Bean
    public static NoOpPasswordEncoder passwordEncoder() {
        return (NoOpPasswordEncoder) NoOpPasswordEncoder.getInstance();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.jdbcAuthentication()
                .dataSource(dataSource);
    }

}
```

> 友情提示：如果胖友想要自定义用户的读取，可以参考[《芋道 Spring Boot 安全框架 Spring Security 入门》](http://www.iocoder.cn/Spring-Boot/Spring-Security/?self)文章。

### 2.5 OAuth2AuthorizationServerConfig

创建 [OAuth2AuthorizationServerConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/src/main/java/cn/iocoder/springboot/lab68/authorizationserverdemo/config/OAuth2AuthorizationServerConfig.java) 配置类，通过 **Spring Security OAuth** 提供**授权服务器**的功能。代码如下：

```text
@Configuration
@EnableAuthorizationServer
public class OAuth2AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    /**
     * 用户认证 Manager
     */
    @Autowired
    private AuthenticationManager authenticationManager;

    /**
     * 数据源 DataSource
     */
    @Autowired
    private DataSource dataSource;

    @Bean
    public TokenStore jdbcTokenStore() {
        return new JdbcTokenStore(dataSource);
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(authenticationManager)
                .tokenStore(jdbcTokenStore());
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer oauthServer) throws Exception {
        oauthServer.checkTokenAccess("isAuthenticated()");
    }

    @Bean
    public ClientDetailsService jdbcClientDetailsService() {
        return new JdbcClientDetailsService(dataSource);
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.withClientDetails(jdbcClientDetailsService());
    }

}
```

> 友情提示：如果胖友看不懂这个配置类，回到[《芋道 Spring Security OAuth2 存储器》](http://www.iocoder.cn/Spring-Security/OAuth2-learning-store/?self)文章复习下。

### 2.6 AuthorizationServerApplication

创建 [AuthorizationServerApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-authorization-server-on-sso/src/main/java/cn/iocoder/springboot/lab68/authorizationserverdemo/AuthorizationServerApplication.java) 类，统一登录系统的启动类。代码如下：

```text
@SpringBootApplication
public class AuthorizationServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(AuthorizationServerApplication.class, args);
    }

}
```

### 2.7 简单测试

执行 AuthorizationServerApplication 启动统一登录系统。下面，我们使用 Postman 模拟一个 Client，**测试我们是否搭建成功**！

`POST` 请求 [http://localhost:8080/oauth/token](http://localhost:8080/oauth/token) 地址，使用密码模式进行**授权**。如下图所示：

![&#x5BC6;&#x7801;&#x6A21;&#x5F0F;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/15.png)

成功获取到访问令牌，成功！

## 3. 搭建 XXX 系统

> 示例代码对应仓库：
>
> * XXX 系统：[`lab-68-demo21-resource-server-on-sso`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-resource-server-on-sso/)

创建 [`lab-68-demo21-resource-server-on-sso`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-resource-server-on-sso/) 项目，搭建 **XXX 系统**，接入**统一登录系统**实现 SSO 功能。

> 友情提示：整个实现代码，和我们前文看到的**资源**服务器是基本一致的。

### 3.1 引入依赖

创建 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/pom.xml) 文件，引入 Spring Security OAuth 依赖。

```text
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>lab-68</artifactId>
        <groupId>cn.iocoder.springboot.labs</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-68-demo21-resource-server</artifactId>

    <properties>
        <!-- 依赖相关配置 -->
        <spring.boot.version>2.2.4.RELEASE</spring.boot.version>
        <!-- 插件相关配置 -->
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.source>1.8</maven.compiler.source>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- 实现对 Spring MVC 的自动配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 实现对 Spring Security OAuth2 的自动配置 -->
        <dependency>
            <groupId>org.springframework.security.oauth.boot</groupId>
            <artifactId>spring-security-oauth2-autoconfigure</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>
    </dependencies>

</project>
```

### 3.2 配置文件

创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-resource-server-on-sso/src/main/resources/application.yml) 配置文件，添加 **SSO** 相关配置：

```text
server:
  port: 9090
  servlet:
    session:
      cookie:
        name: SSO-SESSIONID # 自定义 Session 的 Cookie 名字，防止冲突。冲突后，会导致 SSO 登录失败。

security:
  oauth2:
    # OAuth2 Client 配置，对应 OAuth2ClientProperties 类
    client:
      client-id: clientapp
      client-secret: 112233
      user-authorization-uri: http://127.0.0.1:8080/oauth/authorize # 获取用户的授权码地址
      access-token-uri: http://127.0.0.1:8080/oauth/token # 获取访问令牌的地址
    # OAuth2 Resource 配置，对应 ResourceServerProperties 类
    resource:
      token-info-uri: http://127.0.0.1:8080/oauth/check_token # 校验访问令牌是否有效的地址
```

① `server.servlet.session.cookie.name` 配置项，自定义 Session 的 Cookie 名字，防止冲突。冲突后，会导致 SSO 登录失败。

> 友情提示：具体的值，胖友可以根据自己的喜欢设置。

② `security.oauth2.client` 配置项，OAuth2 Client 配置，对应 [OAuth2ClientProperties](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/oauth2/client/OAuth2ClientProperties.java) 类。在这个配置项中，我们添加了客户端的 `client-id` 和 `client-secret`。

③ `security.oauth2.client.user-authorization-uri` 配置项，获取用户的**授权码**地址。

在访问 XXX 系统需要登录的地址时，Spring Security OAuth 会自动跳转到**统一登录系统**，进行统一登录获取**授权**。

而这里配置的 `security.oauth2.client.user-authorization-uri` 地址，就是之前**授权**服务器的 `oauth/authorize` 接口，可以进行**授权码**模式的授权。

> 友情提示：如果胖友忘记**授权**服务器的 `oauth/authorize` 接口，建议回看下[《芋道 Spring Security OAuth2 入门》](http://www.iocoder.cn/Spring-Security/OAuth2-learning/?self)的[「3. 授权码模式」](http://www.iocoder.cn/Spring-Security/OAuth2-learning-sso/#)小节。

④ `security.oauth2.client.access-token-uri` 配置项，获取**访问令牌**的地址。

在**统一登录系统**完成统一登录并授权后，浏览器会跳转回 XXX 系统的回调地址。在该地址上，会调用**统一登录系统**的 `security.oauth2.client.user-authorization-uri` 地址，通过**授权码**获取到**访问令牌**。

而这里配置的 `security.oauth2.client.user-authorization-uri` 地址，就是之前**授权**服务器的 `oauth/token` 接口。

⑤ `security.oauth2.resource.client.token-info-uri` 配置项，校验**访问令牌**是否有效的地址。

在获取到**访问令牌**之后，每次请求 XXX 系统时，都会调用 **统一登录系统**的 `security.oauth2.resource.client.token-info-uri` 地址，校验访问令牌的有效性，同时返回**用户的基本信息**。

而这里配置的 `security.oauth2.resource.client.token-info-uri` 地址，就是之前**授权**服务器的 `oauth/check_token` 接口。

至此，我们可以发现，Spring Security OAuth 实现的 SSO 单点登录功能，是基于其**授权码模式**实现的。这一点，非常重要，稍后我们演示下会更加容易理解到。

### 3.3 OAuthSsoConfig

创建 [OAuthSsoConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-resource-server-on-sso/src/main/java/cn/iocoder/springboot/lab68/resourceserverdemo/config/OAuthSsoConfig.java) 类，配置接入 SSO 功能。代码如下：

```text
@Configuration
@EnableOAuth2Sso // 开启 Sso 功能
public class OAuthSsoConfig {

}
```

在类上添加 [`@EnableOAuth2Sso`](https://github.com/spring-projects/spring-security-oauth2-boot/blob/master/spring-security-oauth2-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/oauth2/client/EnableOAuth2Sso.java) 注解，声明基于 Spring Security OAuth 的方式接入 SSO 功能。

> 友情提示：想要深入的胖友，可以看看 [SsoSecurityConfigurer](https://github.com/spring-projects/spring-security-oauth2-boot/blob/master/spring-security-oauth2-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/oauth2/client/SsoSecurityConfigurer.java) 类。

### 3.4 UserController

创建 [UserController](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-resource-server-on-sso/src/main/java/cn/iocoder/springboot/lab68/resourceserverdemo/controller/UserController.java) 类，提供获取当前用户的 `/user/info` 接口。代码如下：

```text
@RestController
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/info")
    public Authentication info(Authentication authentication) {
        return authentication;
    }

}
```

### 3.5 ResourceServerApplication

创建 [ResourceServerApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-resource-server-on-sso/src/main/java/cn/iocoder/springboot/lab68/resourceserverdemo/ResourceServerApplication.java) 类，XXX 系统的启动类。代码如下：

```text
@SpringBootApplication
public class ResourceServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ResourceServerApplication.class, args);
    }

}
```

### 3.6 简单测试（第一弹）

执行 ResourceServerApplication 启动 XXX 系统。下面，我们来演示下 SSO 单点登录的过程。

① 使用浏览器，访问 **XXX 系统**的 [http://127.0.0.1:9090/user/info](http://127.0.0.1:9090/user/info) 地址。因为暂未登录，所以被重定向到**统一登录系统**的 [http://127.0.0.1:8080/oauth/authorize](http://127.0.0.1:8080/oauth/authorize) **授权**地址。

又因为在**统一登录系统**暂未登录，所以被重定向到**统一登录系统**的 [http://127.0.0.1:8080/login](http://127.0.0.1:8080/login) **登录**地址。如下图所示：

![&#x767B;&#x5F55;&#x754C;&#x9762;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/21.png)

② 输入用户的账号密码「yunai/1024」，进行**统一登录系统**的登录。登录完成后，进入**统一登录系统**的 [http://127.0.0.1:8080/oauth/authorize](http://127.0.0.1:8080/oauth/authorize) **授权**地址。如下图所示：

![&#x6388;&#x6743;&#x754C;&#x9762;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/22.png)

③ 点击「Authorize」按钮，完成用户的授权。授权完成后，浏览器重定向到 **XXX 系统**的 [http://127.0.0.1:9090/login](http://127.0.0.1:9090/login) **回调**地址。

在 **XX 系统**的回调地址，拿到授权的**授权码**后，会**自动**请求**统一登录系统**，通过**授权码**获取到**访问令牌**。如此，我们便完成了 **XXX 系统** 的登录。

获取授权码完成后，**自动**跳转到登录前的 [http://127.0.0.1:9090/user/info](http://127.0.0.1:9090/user/info) 地址，打印出当前登录的用户信息。如下图所示：

![&#x7528;&#x6237;&#x4FE1;&#x606F;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/23.png)

如此，我们从**统一登录系统**也拿到了**用户信息**。下面，我们来进一步将 Spring Security 的**权限控制**功能来演示下。

### 3.7 SecurityConfig

创建 [SecurityConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-resource-server-on-sso/src/main/java/cn/iocoder/springboot/lab68/resourceserverdemo/config/SecurityConfig.java) 配置类，添加 Spring Security 的功能。代码如下：

```text
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true) // 开启对 Spring Security 注解的方法，进行权限验证。
@Order(101) // OAuth2SsoDefaultConfiguration 使用了 Order(100)，所以这里设置为 Order(101)，防止相同顺序导致报错
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}
```

在类上，增加 [`@EnableGlobalMethodSecurity`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/method/configuration/EnableGlobalMethodSecurity.html) 注解，开启对 Spring Security 注解的方法，进行权限验证。

### 3.8 DemoController

创建 [DemoController](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-68-spring-security-oauth/lab-68-demo21-resource-server-on-sso/src/main/java/cn/iocoder/springboot/lab68/resourceserverdemo/controller/DemoController.java) 类，提供测试权限的功能的接口。代码如下：

```text
@RestController
@RequestMapping("/demo")
public class DemoController {

    @GetMapping("/admin-list")
    @PreAuthorize("hasRole('ADMIN')") // 要求管理员 ROLE_ADMIN 角色
    public String adminList() {
        return "管理员列表";
    }

    @GetMapping("/user-list")
    @PreAuthorize("hasRole('USER')") // 要求普通用户 ROLE_USER 角色
    public String userList() {
        return "用户列表";
    }

}
```

因为当前登录的用户只有 **ROLE\_USE** 角色，所以**可以**访问 `/demo/user-list` 接口，**无法**访问 `/demo/admin-list` 接口。

### 3.9 简单测试（第二弹）

执行 ResourceServerApplication **重启** XXX 系统。下面，我们来演示下 Spring Security 的权限控制功能。

① 使用浏览器，访问 [http://127.0.0.1:9090/demo/user-list](http://127.0.0.1:9090/demo/user-list) 地址，**成功**。如下图所示：

![&#x6210;&#x529F;&#x8BBF;&#x95EE;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/24.png)

② 使用浏览器，访问 [http://127.0.0.1:9090/demo/admin-list](http://127.0.0.1:9090/demo/admin-list) 地址，**失败**。如下图所示：

![&#x5931;&#x8D25;&#x8BBF;&#x95EE;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/02/25.png)

## 666. 彩蛋

至此，我们成功使用 Spring Security OAuth 实现了一个 SSO 单点登录的示例。下图，是 SSO 的整体流程图，胖友可以继续深入理解下：

![SSO &#x6D41;&#x7A0B;&#x56FE;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/02/f616b6e1946340ab63318cfcf269bef6.jpg)

后续，想要深入的胖友，可以看看 Spring Security OAuth 提供的如下两个过滤器：

* [OAuth2ClientContextFilter](https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/client/filter/OAuth2ClientContextFilter.java)
* \[OAuth2ClientAuthenticationProcessingFilter\]\(



