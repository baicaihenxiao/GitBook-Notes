# 闲扯Maven项目代码组织形式

[https://mp.weixin.qq.com/s/lM2dnz32TponWTRmjQstig](https://mp.weixin.qq.com/s/lM2dnz32TponWTRmjQstig)



\1. 代码组织形式

*
  * 1.1 平铺
    * 1.2 父子结构
* \2. 打包问题
*
  * 2.1 继承
    * 2.2 聚合
* \3. 小结

因为最近有小伙伴问到了，所以我想和大家随便扯扯 Maven 项目中代码的组织形式这个问题。

其实也不是啥大问题，但是如果不懂的话，就像雾里看花，始终不能看的明明白白，懂了就像一层窗户纸，捅破就好了。

所以我们就简单扯几句。

### 1. 代码组织形式

首先来说说代码组织形式。

一般来说，就两种比较常见的形式：

* 平铺
* 父子结构

这两种形式松哥在不同的项目中都有遇到过，所以我们就不说孰优孰劣，单纯来说这两种方案。

#### 1.1 平铺

平铺的代码类似下面这样：

```
├── parent
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   └── resources
│       └── test
│           └── java
├── vhr-dao
│   ├── pom.xml
│   ├── src
│   │   ├── main
│   │   │   ├── java
│   │   │   └── resources
│   │   └── test
│   │       └── java
└── vhr-service
    ├── pom.xml
    ├── src
    │   ├── main
    │   │   ├── java
    │   │   └── resources
    │   └── test
    │       └── java
```

如下图：

![Image](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnzXP83XJGOXgLKgUxJWOxicatU8wlvvfpZD9UciaCGTBwy5OHoUCGo3Wg6eicicLBukyc1280AavldXQ/640?wx_fmt=png\&tp=webp\&wxfrom=5\&wx_lazy=1\&wx_co=1)

可以看到，在这种结构下，parent 父工程和各个子工程从代码组织形式上来看都是平级的，都处于同一个目录下。

不过仔细查看 pom.xml 文件，还是能够清晰的看到这三个 module 的父子关系的：

parent：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.javaboy</groupId>
    <artifactId>parent</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>../vhr-dao</module>
        <module>../vhr-service</module>
    </modules>

</project>
```

**可以看到，在指定 module 时，由于 vhr-dao 和 vhr-service 和 parent 的 pom.xml 不在同一个目录下，所以这里使用了相对路径，相对路径的参考依据是 parent 的 pom.xml 文件位置。**

vhr-dao：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>parent</artifactId>
        <groupId>org.javaboy</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../parent/pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>vhr-dao</artifactId>


</project>
```

**可以看到，relativePath 节点中，通过相对路径指定了 parent 的 pom.xml 文件位置，这个相对路径的参考依据是子模块的 pom.xml 文件。**

vhr-service：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>parent</artifactId>
        <groupId>org.javaboy</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../parent/pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>vhr-service</artifactId>


</project>
```

这个和 vhr-dao 的差不多，不赘述。

#### 1.2 父子结构

父子结构则类似于下面这样：

```
├── maven_parent
│   ├── pom.xml
│   ├── vhr-dao
│   │   ├── pom.xml
│   │   └── src
│   │       ├── main
│   │       │   ├── java
│   │       │   └── resources
│   │       └── test
│   │           └── java
│   └── vhr-service
│       ├── pom.xml
│       └── src
│           ├── main
│           │   ├── java
│           │   └── resources
│           └── test
│               └── java
```

如下图：

![Image](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnzXP83XJGOXgLKgUxJWOxicVxjLibtianauJsBKtzHZXTYruOgj76p90IyJSXp00Lj298wfDqz4Nic7w/640?wx_fmt=png\&tp=webp\&wxfrom=5\&wx_lazy=1\&wx_co=1)

这种父子结构的看起来就非常的层次分明了，parent 和各个 module 一眼就能看出来，我们从 GitHub 上下载的很多开源项目如 Shiro，都是这种结构。

不过文件夹的层级并不能说明任何问题，关键还是要看 pom.xml 中的定义，接下来我们就来看看 parent 的 pom.xml 和各个子模块的 pom.xml 有何异同。

maven_parent：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.javaboy</groupId>
    <artifactId>maven_parent</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>vhr-dao</module>
        <module>vhr-service</module>
    </modules>


</project>
```

**和前面不同的是，这里声明 modules 不需要相对路径了（其实还是相对路径，只是不需要 `../` 了），因为各个子模块和 parent 的 pom.xml 文件处于同一目录下。**

vhr-dao：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>maven_parent</artifactId>
        <groupId>org.javaboy</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>vhr-dao</artifactId>


</project>
```

**这里也不需要通过 relativePath 节点去指定 parent 的 pom.xml 文件位置了，因为 parent 的 pom.xml 和各个子模块处于同一目录下。**

vhr-service：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>maven_parent</artifactId>
        <groupId>org.javaboy</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>vhr-service</artifactId>


</project>
```

### 2. 打包问题

#### 2.1 继承

有的时候，单纯只是想通过 parent 来统一管理不同的项目的依赖，并非一个聚合项目。

这个时候只需要去掉 parent 的 pom.xml 中的 modules 节点及其中的内容即可，这样就不是聚合工程了，各个子模块也可以独立打包。

#### 2.2 聚合

当然很多情况我们是聚合工程。

聚合工程的话，一般松哥是建议大家从 parent 处统一进行打包：

![Image](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnzXP83XJGOXgLKgUxJWOxic2SicVAC8Wp4eKmHKlxibbnk7OGqhY97a7cOp9bAyOdeeZ0lUibSQ8yNgA/640?wx_fmt=png\&tp=webp\&wxfrom=5\&wx_lazy=1\&wx_co=1)

这样可以确保打包到的是最新的代码。

当然还有另外一种操作流程：

1. 首先将 parent 安装到本地仓库。
2. 然后分别将 model、dao 以及 service 等模块安装到本地仓库。
3. 最后 web 模块就可以独立打包了。

如果使用这种操作流程，需要注意一点，就是每个模块代码更新之后，要及时安装到本地仓库，否则当 web 模块独立打包时，用到的其他模块就不是最新的代码。

### 3. 小结

好啦，几个 Maven 中的小问题，窗户纸捅破了就豁然开朗啦～
