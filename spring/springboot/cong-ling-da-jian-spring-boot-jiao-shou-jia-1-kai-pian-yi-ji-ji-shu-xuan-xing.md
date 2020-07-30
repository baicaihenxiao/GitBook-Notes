# 从零搭建Spring Boot脚手架（1）：开篇以及技术选型

[https://mp.weixin.qq.com/s/k0faB9xElGpCyLrJfGc7uQ](https://mp.weixin.qq.com/s/k0faB9xElGpCyLrJfGc7uQ)

### 1. 前言

目前**Spring Boot**已经成为主流的**Java Web**开发框架，熟练掌握**Spring Boot**并能够根据业务来定制**Spring Boot**成为一个**Java**开发者的必备技巧，但是总是零零碎碎不够系统，所以萌生了从零搭建一个后端脚手架的想法。并把这个过程中的细节思路和之前的一些文章结合起来展现给大家，希望能够实实在在帮助学习**Spring Boot**的同学，当然能力有限如果有不足之处还请多多指教。

### 2. 面向的群体

首先，这个定位不是完全没有接触过**Spring Boot**的初学者，因为**Spring Boot**的简单入门并不是特别难，找一些其他大佬的入门教程学习一阵就可以很快的入门；而是面向具有**Spring Boot**的学习经验和不够熟练的同学们，同时提供一些可以开箱即用的解决方案到实际开发中。

### 3. 项目结构介绍

其实我不太喜欢那种相互依赖整了好几个模块，**DAO**、**Service**、**Controller**各搞一个层，然后层层依赖。对于单体项目来说这种结构把简单的事情复杂化了，容易导致依赖管理混乱。所以一般的简单项目我都建议采用下面的结构：

项目总体结构

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/30/640-20200730192133404-192133.png)

**kono-dependencies**是一个依赖版本管理的模块，负责**kono-app**所有的依赖版本、依赖选型的管理。原则上**kono-app**所有的依赖都应该来自**kono-dependencies**而且版本从**kono-dependencies**继承，这样能做到依赖版本的集中控制，使得技术选型和兼容性得到保证。

以**Maven**为例，**kono-dependencies**只会包含一个**pom.xml**，而且打包方式`packaging`只能是`pom`。所有的依赖都被`dependencyManagement`管理。

```text
<groupId>cn.felord</groupId>
<artifactId>kono-dependencies</artifactId>
<version>1.0.0.RELEASE</version>
<!--打包方式-->
<packaging>pom</packaging>

<dependencyManagement>
   <!--被管理的依赖-->
</dependencyManagement>
```

这里有一个小技巧，我们把**Spring Boot**的父依赖加入管理，就等于把项目的**Spring Boot**所有的官方**starter**纳入了管理：

```text
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>${springboot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
         <!-- 其它依赖 -->
    </dependencies>
</dependencyManagement>
```

> 当然如果有业务需要可以分更多的模块，但是依赖管理一定要清晰、可控。

### 4. 版本号

版本号的规则也是很有学问的。这里我选用了最容易理解的方式，也是**Spring Boot**采用的版本号命名风格。

Spring Boot版本号风格

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/30/640-20200730192133956-192134.png)

* **Major** 主版本号，当有非兼容性的变更时，递增主版本号。
* **Minor** 次版本号，当以可向后兼容的方式增加了功能时，递增次的版本号。
* **Patch** 补丁版本号，当有向后兼容的 bug 修复时，递增补丁版本号。
* **Label** 标记，用来区分开发版、快照版、里程碑版、正式发行版。

### 5. 技术选型

以下都是 Java 技术栈特定场景下的常用选择：

* springboot 基础整合框架
* servlet4 web 标准
* undertow 或者 tomcat web 容器
* spring cache 缓存抽象层
* spring security 安全框架
* json web token 安全框架**token**技术
* mybatis plus 3 **ORM**增强
* spring data jpa \(选\)
* redis 缓存中间件
* mysql 数据库
* mapstruct bean 转换器，编译期使用
* lombok bean 简化工具
* swagger2 文档（开发测试）
* docker 容器技术

在一开始，这里面的一些技术并不会集成进去，随着迭代会在合适的时机加入它们，甚至会加入这里面没有的技术栈。

### 6. 最后

通过从零搭建脚手架的过程您可以循序渐进的学到如何整合一些功能到项目中，同时还能看到一些实际开发中才能遇到的一些问题以及解决这些问题的思路。同时如果在这个过程中您有好的建议和问题也可以和我进行沟通，感谢持续关注:**码农小胖哥**，共同提高，我们下一篇见。

