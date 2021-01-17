# SpringBoot 项目模板：摆脱步步搭建



> [https://www.cnblogs.com/davenkin/p/spring-boot-template.html](https://www.cnblogs.com/davenkin/p/spring-boot-template.html)
>
> [https://mp.weixin.qq.com/s?\_\_biz=MzUxOTc4NjEyMw==&mid=2247499947&idx=2&sn=b218be474b36544fd595a91882414bdd&chksm=f9f6db4fce815259d4f1b1b40d1a1d0c0f86774b93423afbda1d11fc0540d5df7853727f7512&mpshare=1&scene=1&srcid=0117pdwv6f7dnDRAifX1hs1N&sharer\_sharetime=1610881249070&sharer\_shareid=393f249533d421d13c2402bd43e74356\#rd](https://mp.weixin.qq.com/s?__biz=MzUxOTc4NjEyMw==&mid=2247499947&idx=2&sn=b218be474b36544fd595a91882414bdd&chksm=f9f6db4fce815259d4f1b1b40d1a1d0c0f86774b93423afbda1d11fc0540d5df7853727f7512&mpshare=1&scene=1&srcid=0117pdwv6f7dnDRAifX1hs1N&sharer_sharetime=1610881249070&sharer_shareid=393f249533d421d13c2402bd43e74356#rd)

* [**前言**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**第一步：从写好README开始**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**一键式本地构建**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**目录结构**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**基于业务分包**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**自动化测试分类**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**日志处理**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**异常处理**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**后台任务与分布式锁**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**统一代码风格**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**静态代码检查**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**健康检查**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**API文档**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**数据库迁移**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**多环境构建**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**CORS**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [**总结**](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)

[![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/01/17/640-190959.jpg)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)

## [前言](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

在我的工作中，我从零开始搭建了不少软件项目，其中包含了基础代码框架和持续集成基础设施等，这些内容在敏捷开发中通常被称为“第0个迭代”要做的事情。但是，当项目运行了一段时间之后再来反观，我总会发现一些不足的地方，要么测试分类没有分好，要么基本的编码架子没有考虑周全。

另外，我在工作中也会接触到很多既有项目，公司内部和外部的都有，多数项目的编码实践我都是不满意的。比如，我曾经新加入一个项目的时候，前前后后请教了3位同事才把该项目在本地运行起来；又比如在另一项目中，我发现前端请求对应的Java类命名规范不统一，有被后缀为Request的，也有被后缀为Command的。

再者，工作了这么多年之后，我越来越发现基础知识以及系统性学习的重要性。诚然，技术框架的发展使得我们可以快速地实现业务功能，但是当软件出了问题之后有时却需要将各方面的知识融会贯通并在大脑里综合反应才能找到解决思路。

基于以上，我希望整理出一套公共性的项目模板出来，旨在尽量多地包含日常开发之所需，减少开发者的重复性工作以及提供一些最佳实践。对于后端开发而言，我选择了当前被行业大量使用的Spring Boot，基于此整理出了一套公共的、基础性的实践方式，在结合了自己的经验以及其他项目的优秀实践之后，总结出本文以飨开发者。

**本文以一个简单的电商订单系统为例，源代码请访问：**

> git clone [https://github.com/e-commerce-sample/order-backend](https://github.com/e-commerce-sample/order-backend) git checkout a443dace

所使用的技术栈主要包括：Spring Boot、Gradle、MySQL、Junit 5、Rest Assured、Docker等。

## [第一步：从写好README开始](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

一份好的README可以给人以项目全景概览，可以使新人快速上手项目，可以降低沟通成本。同时，README应该简明扼要，条理清晰，建议包含以下方面：

* 项目简介：用一两句话简单描述该项目所实现的业务功能；
* 技术选型：列出项目的技术栈，包括语言、框架和中间件等；
* 本地构建：列出本地开发过程中所用到的工具命令；
* 领域模型：核心的领域概念，比如对于示例电商系统来说有Order、Product等；
* 测试策略：自动化测试如何分类，哪些必须写测试，哪些没有必要写测试；
* 技术架构：技术架构图；
* 部署架构：部署架构图；
* 外部依赖：项目运行时所依赖的外部集成方，比如订单系统会依赖于会员系统；
* 环境信息：各个环境的访问方式，数据库连接等；
* 编码实践：统一的编码实践，比如异常处理原则、分页封装等；
* FAQ：开发过程中常见问题的解答。

需要注意的是，README中的信息可能随着项目的演进而改变（比如引入了新的技术栈或者加入了新的领域模型），因此也是需要持续更新的。虽然我们知道，软件文档的一个痛点便是无法与项目实际进展保持同步，但是就README这点信息来讲，还是建议开发者们不要吝啬那一点点敲键盘的时间。

此外，除了保持README的持续更新，一些重要的架构决定可以通过示例代码的形式记录在代码库中，新开发者可以通过直接阅读这些示例代码快速了解项目的通用实践方式以及架构选择，请参考:

> [https://www.thoughtworks.com/radar/techniques/lightweight-architecture-decision-records](https://www.thoughtworks.com/radar/techniques/lightweight-architecture-decision-records)

## [一键式本地构建](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

为了避免诸如前文中所提到的“请教了3位同事才本地构建成功”的尴尬，为了减少“懒惰”的程序员们的手动操作，也为了为所有开发者提供一种一致的开发体验，我们希望用一个命令就可以完成所有的事情。这里，对于不同的场景我总结出了以下命令：

* 生成IDE工程：idea.sh，生成IntelliJ工程文件并自动打开IntelliJ
* 本地运行：run.sh，本地启动项目，自动启动本地数据库，监听调试端口5005
* 本地构建：local-build.sh，只有本地构建成功才能提交代码

以上3个命令基本上可以完成日常开发之所需，此时，对于新人的开发流程大致为：

* 拉取代码；
* 运行idea.sh，自动打开IntelliJ；
* 编写代码，包含业务代码和自动化测试；
* 运行run.sh，进行本地调试或必要的手动测试\(本步骤不是必需\)；
* 运行local-build.sh，完成本地构建；
* 再次拉取代码，保证local-build.sh成功，提交代码。

事实上，这些命令脚本的内容非常简单，比如run.sh文件内容为：

```text
#!/usr/bin/env bash./gradlew clean bootRun
```

然而，这种显式化的命令却可以减少新人的恐惧感，因为他们只需要知道运行这3个命令就可以搞开发了。另外，一个小小的细节：本地构建的local-build.sh命令本来可以重命名为更简单的build.sh，但是当我们在命令行中使用Tab键自动补全的时候，会发现自动补全到了build目录，而不是build.sh命令，并不方便，因此命名为了local-build.sh。

细节虽小，但是却体现了一个宗旨，即我们希望给开发者一种极简的开发体验，我把这些看似微不足道的东西称作是对程序员的“人文关怀”。

## [目录结构](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

Maven所提倡的目录结构当前已经成为事实上的行业标准，Gradle在默认情况下也采用了Maven的目录结构，这对于多数项目来说已经足够了。此外，除了Java代码，项目中还存在其他类型的文件，比如Gradle插件的配置、工具脚本和部署配置等。无论如何，项目目录结构的原则是简单而有条理，不要随意地增加多余的文件夹，并且也需要及时重构。

在示例项目中，顶层只有2个文件夹，一个是用于放置Java源代码和项目配置的src文件夹，另一个是用于放置所有Gradle配置的gradle文件夹，此外，为了方便开发人员使用，将上文提到的3个常用脚本直接放到根目录下：

```text
└── order-backend
    ├── gradle // 文件夹，用于放置所有Gradle配置
    ├── src // 文件夹，Java源代码
    ├── idea.sh //生成IntelliJ工程
    ├── local-build.sh // 提交之前的本地构建
    └── run.sh // 本地运行
```

对于gradle而言，我们刻意地将Gradle插件脚本与插件配置放到了一起，比如Checkstyle：

```text
├── gradle
│   ├── checkstyle
│   │   ├── checkstyle.gradle
│   │   └── checkstyle.xml
```

事实上，在默认情况下Checkstyle插件会从项目根目录下的config目录查找checkstyle.xml配置文件，但是这一方面增加了多余的文件夹，另一方面与该插件相关的设施分散在了不同的地方，违背了广义上的内聚原则。

## [基于业务分包](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

早年的Java分包方式通常是基于技术的，比如与domain包平级的有controller包、service包和infrastructure包等。这种方式当前并不被行业所推崇，而是应该首先基于业务分包。

比如，在订单示例项目中，有两个重要的领域对象Order和Product（在DDD中称为聚合根），所有的业务都围绕它们展开，因此分别创建order包和product包，再分别在包下创建与之相关的各个子包。此时的order包如下：

```text
├── order
│   ├── OrderApplicationService.java
│   ├── OrderController.java
│   ├── OrderNotFoundException.java
│   ├── OrderRepository.java
│   ├── OrderService.java
│   └── model
│       ├── Order.java
│       ├── OrderFactory.java
│       ├── OrderId.java
│       ├── OrderItem.java
│       └── OrderStatus.java
```

可以看到，在order包下我们直接放置了OrderController和OrderRepository等类，而没有必要再为这些类划分单独的子包。而对于领域模型Order来讲，由于包含了多个对象，因此基于内聚性原则将它们归到model包中。但是这并不是一个必须，如果业务足够简单，我们甚至可以将所有类直接放到业务包下，product包便是如此：

```text
└── product
    ├── Product.java
    ├── ProductApplicationService.java
    ├── ProductController.java
    ├── ProductId.java
    └── ProductRepository.java
```

在编码实践中，我们总是基于一个业务用例来实现代码，在技术分包场景下，我们需要在分散的各包中来回切换，增加了代码导航的成本；另外，代码提交的变更内容也是散落的，在查看代码提交历史时，无法直观的看出该次提交是关于什么业务功能的。

在业务分包下，我们只需要在单个统一的包下修改代码，减少了代码导航成本；另外一个好处是，如果哪天我们需要将某个业务迁移到另外的项目（比如识别出了独立的微服务），那么直接整体移动业务包即可。

当然，基于业务分包并不意味着所有的代码都必须囿于业务包下，这里的逻辑是：优先进行业务分包，然后对于一些不隶属于任何业务的代码可以单独分包，比如一些util类、公共配置等。比如我们依然可以创建一个common包，下面放置了Spring公共配置、异常处理框架和日志等子包：

```text
└── common
    ├── configuration
    ├── exception
    ├── loggin
    └── utils
```

## [自动化测试分类](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

在当前的微服务和前后端分离的开发模式下，后端项目仅提供纯粹的业务API，而不包含UI逻辑，因此后端项目不会再包含诸如WebDriver的重量级端到端测试。同时，后端项目作为向外提供业务功能的独立运行单元，在API级别也应该有相应的测试。

此外，程序中有些框架性代码，要么是诸如Controller之类的技术性框架代码，要么是基于某种架构风格的代码（比如DDD实践中的ApplicationService），这些代码一方面并不包含业务逻辑，一方面是很薄的一个抽象层（即实现相对简单），用单元测试来覆盖显得没有必要，因此笔者的观点是可以不为此编写单独的单元测试。

再者，程序中有些重要的组件性代码，比如访问数据库的Repository或者分布式锁，使用单元测试实际上“测不到点上”，而使用API测试又显得在分类逻辑上不合理，为此我们可以专门创建一种测试类型谓之组件测试。

基于以上，我们可以对自动化测试做个分类：

* 单元测试：核心的领域模型，包括领域对象\(比如Order类\)，Factory类，领域服务类等；
* 组件测试：不适合写单元测试但是又必须测试的类，比如Repository类，在有些项目中，这种类型测试也被称为集成测试；
* API测试：模拟客户端测试各个API接口，需要启动程序。

Gradle在默认情况下只提供src/test/java目录用于测试，对于以上3种类型的测试，我们需要将它们分开以便于管理（也是职责分离的体现）。为此，可以通过Gradle提供的SourceSets对测试代码进行分类：

```text
sourceSets {
    componentTest {
        compileClasspath += sourceSets.main.output + sourceSets.test.output
        runtimeClasspath += sourceSets.main.output + sourceSets.test.output
    }

    apiTest {
        compileClasspath += sourceSets.main.output + sourceSets.test.output
        runtimeClasspath += sourceSets.main.output + sourceSets.test.output
    }
}
```

到此，3种类型的测试可以分别编写在以下目录：

* 单元测试：src/test/java
* 组件测试：src/componentTest/java
* API测试：src/apiTest/java

需要注意的是，这里的API测试更多强调的是对业务功能的测试，有些项目中可能还会存在契约测试和安全测试等，虽然从技术上讲都是对API的访问，但是这些测试都是单独的关注点，因此建议分开对待。

值得一提的是，由于组件测试和API测试需要启动程序，也即需要准备好本地数据库，我们采用了Gradle的docker-compose插件\(或者jib插件\)，该插件会在运行测试之前自动运行Docker容器（比如MySQL）：

```text
apply plugin: 'docker-compose'

dockerCompose {
    useComposeFiles = ['docker/mysql/docker-compose.yml']
}

bootRun.dependsOn composeUp
componentTest.dependsOn composeUp
apiTest.dependsOn composeUp
```

更多的测试分类配置细节，比如JaCoCo测试覆盖率配置等，请参考本文的示例项目代码。对Gradle不熟悉的读者可以参考：

> [https://www.cnblogs.com/CloudTeng/p/3417762.html](https://www.cnblogs.com/CloudTeng/p/3417762.html)

## [日志处理](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

在日志处理中，除了完成基本配置外，还有2个需要考虑的点：

1、在日志中加入请求标识，便于链路追踪。在处理一个请求的过程中有时会输出多条日志，如果每条日志都共享统一的请求ID，那么在日志追踪时会更加方便。此时，可以使用Logback原生提供的MDC\(Mapped Diagnostic Context\)功能，创建一个RequestIdMdcFilter：

```text
 protected void doFilterInternal(HttpServletRequest request,HttpServletResponse response,FilterChain filterChain)
            throws ServletException, IOException {
    //request id in header may come from Gateway, eg. Nginx
    String headerRequestId = request.getHeader(HEADER_X_REQUEST_ID);
    MDC.put(REQUEST_ID, isNullOrEmpty(headerRequestId) ? newUuid() : headerRequestId);
    try {
        filterChain.doFilter(request, response);
    } finally {
        clearMdc();
    }
    }
```

2、集中式日志管理，在多节点部署的场景下，各个节点的日志是分散的，为此可以引入诸如ELK之类的工具将日志统一输出到ElasticSearch中。本文的示例项目使用了RedisAppender将日志输出到Logstash：

```text
<appender name="REDIS" class="com.cwbase.logback.RedisAppender">
    <tags>ecommerce-order-backend-${ACTIVE_PROFILE}</tags>
    <host>elk.yourdomain.com</host>
    <port>6379</port>
    <password>whatever</password>
    <key>ecommerce-ordder-log</key>
    <mdc>true</mdc>
    <type>redis</type>
</appender>
```

当然，统一日志的方案还有很多，比如Splunk和Graylog等。

## [异常处理](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

在设计异常处理的框架时，需要考虑以下几点：

* 向客户端提供格式统一的异常返回
* 异常信息中应该包含足够多的上下文信息，最好是结构化的数据以便于客户端解析
* 不同类型的异常应该包含唯一标识，以便客户端精确识别

异常处理通常有两种形式，一种是层级式的，即每种具体的异常都对应了一个异常类，这些类最终继承自某个父异常；另一种是单一式的，即整个程序中只有一个异常类，再以一个字段来区分不同的异常场景。

层级式异常的好处是能够显式化异常含义，但是如果层级设计不好可能导致整个程序中充斥着大量的异常类；单一式的好处是简单，而其缺点在于表意性不够。

本文的示例项目使用了层级式异常，所有异常都继承自一个AppException：

```text
public abstract class AppException extends RuntimeException {
    private final ErrorCode code;
    private final Map<String, Object> data = newHashMap();
}
```

这里，ErrorCode枚举中包含了异常的唯一标识、HTTP状态码以及错误信息；而data字段表示各个异常的上下文信息。

在示例系统中，在没有找到订单时抛出异常：

```text
public class OrderNotFoundException extends AppException {
    public OrderNotFoundException(OrderId orderId) {
        super(ErrorCode.ORDER_NOT_FOUND, ImmutableMap.of("orderId", orderId.toString()));
    }
}
```

在返回异常给客户端时，通过一个ErrorDetail类来统一异常格式：

```text
public final class ErrorDetail {
    private final ErrorCode code;
    private final int status;
    private final String message;
    private final String path;
    private final Instant timestamp;
    private final Map<String, Object> data = newHashMap();
}
```

最终返回客户端的数据为：

```text
{
  requestId: "d008ef46bb4f4cf19c9081ad50df33bd",
  error: {
    code: "ORDER_NOT_FOUND",
    status: 404,
    message: "没有找到订单",
    path: "/order",
    timestamp: 1555031270087,
    data: {
      orderId: "123456789"
    }
  }
}
```

可以看到，ORDER\_NOT\_FOUND与data中的数据结构是一一对应的，也即对于客户端来讲，如果发现了ORDER\_NOT\_FOUND，那么便可确定data中一定存在orderId字段，进而完成精确的结构化解析。

## [后台任务与分布式锁](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

除了即时完成客户端的请求外，系统中通常会有一些定时性的例行任务，比如定期地向用户发送邮件或者运行数据报表等；另外，有时从设计上我们会对请求进行异步化处理。此时，我们需要搭建后台任务相关基础设施。Spring原生提供了任务处理\(TaskExecutor\)和任务计划\(TaskSchedulor\)机制；而在分布式场景下，还需要引入分布式锁来解决并发冲突，为此我们引入一个轻量级的分布式锁框架ShedLock。

启用Spring任务配置如下：

```text
@Configuration
@EnableAsync
@EnableScheduling
public class SchedulingConfiguration implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(newScheduledThreadPool(10));
    }

    @Bean(destroyMethod = "shutdown")
    @Primary
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(10);
        executor.setTaskDecorator(new LogbackMdcTaskDecorator());
        executor.initialize();
        return executor;
    }

}
```

然后配置Shedlock：

```text
@Configuration
@EnableSchedulerLock(defaultLockAtMostFor = "PT30S")
public class DistributedLockConfiguration {

    @Bean
    public LockProvider lockProvider(DataSource dataSource) {
        return new JdbcTemplateLockProvider(dataSource);
    }

    @Bean
    public DistributedLockExecutor distributedLockExecutor(LockProvider lockProvider) {
        return new DistributedLockExecutor(lockProvider);
    }

}
```

实现后台任务处理：

```text
 @Scheduled(cron = "0 0/1 * * * ?")
    @SchedulerLock(name = "scheduledTask", lockAtMostFor = THIRTY_MIN, lockAtLeastFor = ONE_MIN)
    public void run() {
        logger.info("Run scheduled task.");
    }
```

为了支持代码直接调用分布式锁，基于Shedlock的LockProvider创建DistributedLockExecutor：

```text
public class DistributedLockExecutor {
    private final LockProvider lockProvider;

    public DistributedLockExecutor(LockProvider lockProvider) {
        this.lockProvider = lockProvider;
    }

    public <T> T executeWithLock(Supplier<T> supplier, LockConfiguration configuration) {
        Optional<SimpleLock> lock = lockProvider.lock(configuration);
        if (!lock.isPresent()) {
            throw new LockAlreadyOccupiedException(configuration.getName());
        }

        try {
            return supplier.get();
        } finally {
            lock.get().unlock();
        }
    }

}
```

使用时在代码中直接调用：

```text
public String doBusiness() {
    return distributedLockExecutor.executeWithLock(() -> "Hello World.",
            new LockConfiguration("key", Instant.now().plusSeconds(60)));
}
```

本文的示例项目使用了基于JDBC的分布式锁，事实上任何提供原子操作的机制都可用于分布式锁，Shedlock还提供基于Redis、ZooKeeper和Hazelcast等的分布式锁实现机制。

## [统一代码风格](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

除了Checkstyle统一代码格式之外，项目中有些通用的公共的编码实践方式也需要在整个开发团队中进行统一，包括但不限于以下方面：

* 客户端的请求数据类统一使用相同后缀，比如Command
* 返回给客户端的数据统一使用相同后缀，比如Represetation
* 统一对请求处理的流程框架，比如采用传统的3层架构或者DDD战术模式
* 提供一致的异常返回（请参考“异常处理”小节）
* 提供统一的分页结构类
* 明确测试分类以及统一的测试基础类（请参考“自动化测试分类”小节）

## [静态代码检查](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

静态代码检查主要包含以下Gradle插件，具体配置请参考本文示例代码：

* Checkstyle：用于检查代码格式，规范编码风格
* Spotbugs：Findbugs的继承者
* Dependency check：OWASP提供的Java类库安全性检查
* Sonar：用于代码持续改进的跟踪

## [健康检查](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

健康检查主要用于以下场景：

* 我们希望初步检查程序是否运行正常
* 有些负载均衡软件会通过一个健康检查URL判断节点的可达性

此时，可以实现一个简单的API接口，该接口不受权限管控，可以公开访问。如果该接口返回HTTP的200状态码，便可初步认为程序运行正常。此外，我们还可以在该API中加入一些额外的信息，比如提交版本号、构建时间、部署时间等。

启动本文的示例项目：

```text
./run.sh
```

然后访问健康检查API：[http://localhost:8080/about，结果如下：](http://localhost:8080/about，结果如下：)

```text
{
  requestId: "698c8d29add54e24a3d435e2c749ea00",
  buildNumber: "unknown",
  buildTime: "unknown",
  deployTime: "2019-04-11T13:05:46.901+08:00[Asia/Shanghai]",
  gitRevision: "unknown",
  gitBranch: "unknown",
  environment: "[local]"
}
```

以上接口在示例项目中用了一个简单的Controller实现，事实上Spring Boot的Acuator框架也能够提供相似的功能。

## [API文档](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

软件文档的难点不在于写，而在于维护。多少次，当我对照着项目文档一步一步往下走时，总得不到正确的结果，问了同事之后得到回复“哦，那个已经过时了”。本文示例项目所采用的Swagger在一定程度上降低了API维护的成本，因为Swagger能自动识别代码中的方法参数、返回对象和URL等信息，然后自动地实时地创建出API文档。

配置Swagger如下：

```text
@Configuration
@EnableSwagger2
@Profile(value = {"local", "dev"})
public class SwaggerConfiguration {

    @Bean
    public Docket api() {
        return new Docket(SWAGGER_2)
                .select()
                .apis(basePackage("com.ecommerce.order"))
                .paths(any())
                .build();
    }
}
```

启动本地项目，访问[http://localhost:8080/swagger-ui.html：](http://localhost:8080/swagger-ui.html：)

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/01/17/640-20210117190958876-190959.jpg)

## [数据库迁移](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

在传统的开发模式中，数据库由专门的运维团队或者DBA来维护，要对数据库进行修改需要向DBA申请，告之迁移内容，最后由DBA负责数据库变更实施。在持续交付和DevOps运动中，这些工作逐步提前到开发过程，当然并不是说不需要DBA了，而是这些工作可以由开发者和运维人员一同完成。

另外，在微服务场景下，数据库被包含在单个服务的边界之内，因此基于内聚性原则（咦，这好像是本文第三次提到内聚原则了，可见其在软件开发中的重要性），数据库的变更最好也与项目代码一道维护在代码库中。

本文的示例项目采用了Flyway作为数据库迁移工具，加入了Flyway依赖后，在src/main/sources/db/migration目录下创建迁移脚本文件即可：

```text
resources/
├── db
│   └── migration
│       ├── V1__init.sql
│       └── V2__create_product_table.sql
```

迁移脚本的命名需要遵循一定的规则以保证脚本执行顺序，另外迁移文件生效之后不要任意修改，因为Flyway会检查文件的checksum，如果checksum不一致将导致迁移失败。

## [多环境构建](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

在软件的开发流程中，我们需要将软件部署到多个环境，经过多轮验证后才能最终上线。在不同的阶段中，软件的运行态可能是不一样的，比如本地开发时可能将所依赖的第三方系统stub掉；持续集成构建时可能使用的是测试用的内存数据库等等。为此，本文的示例项目推荐采用以下环境：

* local：用于开发者本地开发
* ci：用于持续集成
* dev：用于前端开发联调
* qa：用于测试人员
* uat：类生产环境，用于功能验收\(有时也称为staging环境\)
* prod：正式的生产环境

## [CORS](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

在前后端分离的系统中，前端单独部署，有时连域名都和后端不同，此时需要进行跨域处理。传统的做法可以通过JSONP，但这是一种比较“trick”的做法，当前更通用的实践是采用CORS机制，在Spring Boot项目中，启用CORS配置如下：

```text
@Configuration
public class CorsConfiguration {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**");
            }
        };
    }
}
```

对于使用Spring Security的项目，需要保证CORS工作于Spring Security的过滤器之前，为此Spring Security专门提供了相应配置：

```text
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            // by default uses a Bean by the name of corsConfigurationSource
            .cors().and()
            ...
    }

    @Bean
    CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("https://example.com"));
        configuration.setAllowedMethods(Arrays.asList("GET","POST"));
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

常用第三方类库

这里列出一些比较常见的第三方库，开发者们可以根据项目所需引入：

* Guava：来自Google的常用类库
* Apache Commons：来自Apache的常用类库
* Mockito：主要用于单元测试的mock
* DBUnit：测试中管理数据库测试数据
* Rest Assured：用于Rest API测试
* Jackson 2：Json数据的序列化和反序列化
* jjwt：Jwt token认证
* Lombok：自动生成常见Java代码，比如equals\(\)方法，getter和setter等；
* Feign：声明式Rest客户端
* Tika：用于准确检测文件类型
* itext：生成Pdf文件等
* zxing：生成二维码
* Xstream：比Jaxb更轻量级的XML处理库

## [总结](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

本文通过一个示例项目谈及到了项目之初开发者搭建后端工程的诸多方面，其中的绝大多数实践均在笔者的项目中真实落地。读完本文之后你可能会发现，文中的很多内容都是很基础很简单的。

没错，的确没有什么难的东西，但是要系统性地搭建好后端项目的基础框架却不见得是每个开发团队都已经做到的事情，而这恰恰是本文的目的。

最后，需要提醒的是，本文提到的实践方式只是一个参考，一方面依然存在考虑不周的地方，另一方面示例项目中用到的技术工具还存在其他替代方案，请根据自己项目的实际情况进行取舍。

