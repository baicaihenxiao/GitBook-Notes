# 浅谈之UT和Mock 你得做一个合格的工程师

[https://mp.weixin.qq.com/s/pPJO\_qVBJibkMN6OG7d-Bg](https://mp.weixin.qq.com/s/pPJO_qVBJibkMN6OG7d-Bg)

作者 \| S.L

来源 \| [https://elsef.com/2018/04/05/%E6%B5%85%E8%B0%88%E4%B9%8BUT%E5%92%8CMock/](https://elsef.com/2018/04/05/%E6%B5%85%E8%B0%88%E4%B9%8BUT%E5%92%8CMock/)

## 关于测试

### **1 测试都包括哪些**

广义的测试包括 UT、IT、压力测试、硬件测试等等，这里重点讨论 Unit Test 即单元测试。

### **2 啥是 UT**

单元测试（又称为模块测试, Unit Testing）是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作。程序单元是应用的最小可测试部件。在过程化编程中，一个单元就是单个程序、函数、过程等；对于面向对象编程，最小单元就是方法，包括基类（超类）、抽象类、或者派生类（子类）中的方法。

简而言之就是覆盖你的代码的一些测试用例，不依赖于任何第三方的服务依赖，如 HTTP 接口、数据库连接等，只测试功能不依赖于环境，在任何时候人和机器上都可以 Pass。

### **3 为什么要写 UT**

让你的代码质量更可靠&让你对代码结构更加敏感&迫使你写更优质的代码&…

### **4 为什么不写 UT**

！${为什么要写 UT}

### **5 什么在阻止你写 UT**

> * 代码本身的原因
> * * 如果代码复杂度较高还缺少必要的抽象和拆分，就会让人对写 UT 望而生畏。
> * 编码工作量的原因
> * * 无论是用什么样的单元测试框架，最后写出来的单元测试代码量也比业务代码只多不少，在不作弊的前提下要保证相关的测试覆盖率，大概要三倍源码左右的工作量。
> * 难以维护的原因

更多的代码量，加上单测代码并不像业务代码那样直观，还有对单测代码可读性不重视的坏习惯，导致最终呈现出来的单测代码难以阅读，要维护更是难上加难。

### **6 合格的 UT 什么样**

至少要满足：

> * 测试的是一个代码单元内部的逻辑，而不是各模块之间的交互。
> * 无依赖，不需要实际运行环境就可以测试代码。
> * 运行效率高，可以随时执行。

## Java 如何写 UT

Java 开发一般都是用 JUnit 或 TestNG，我们大多人还是使用 JUnit4。本文不讨论语法，只介绍一般性的使用规范。

### **7 命名**

可以参考 7 Popular Unit Test Naming \( [https://dzone.com/articles/7-popular-unit-test-naming](https://dzone.com/articles/7-popular-unit-test-naming) \)

### **8 Assertion**

任何一个 UT 中需要至少包含一个 assert，用 System.out.println\(\)来验证结果不符合 UT 的规范，一般都是验证方法的返回结果，如 assertEquals\(200, statusCode\)而不是 System.out.println\(200==statusCode\)。

Assertion 只能保证走过的分支的结果是否正确，无法保证一定是走过了某些分支。

### **9 为啥要 Mock**

不用 Mock 我们自己也能实现测试（如匿名类），只不过对代码的要求非常高

### **10 Mock 框架**

一些常用的 mock 库包括 Mockito、JMockIt、EasyMock、PowerMock…没有优劣没有好坏，只有合适与否。

比如我个人比较喜欢 Mockito：

> * 第一它相对于其他几个老牌库来说比较新并且更新活跃，在 github 中引用的也最多
> * 第二它的 fluent API 风格的代码可读性很高跟 JDK8 的 Stream 风格很像
> * 第三它抽象出测试中的经典概念，如 when\(\).thenReturn\(\)、doThrow\(\).when\(\)、verify\(\)、times\(\)、never\(\)以及各种注解很容易理解

### **11 什么样的方法需要 mock**

> * 任何被非本类的功能均需要 mock，如数据库访问、RPC 接口、外部引入的 jar 包等
> * 环境变量、系统属性和方法
> * 测试只测试当前类当前方法的功能，依赖方的功能由依赖方的 UT 来保证正确性，本层不负责验证
> * mock 本质上是一个 proxy，在需要提供功能的时候由开发者提供“伪实现”

### **12 什么样的方法不需要 mock**

> * 本类的需要测试的方法依赖的同类方法，该方法的正确性由该方法自身的 UT 来保证
> * 静态方法，静态方法由自身的 UT 来保证功能的正确性
> * protected 方法是可以测试的，只要测试代码类和要测试的类在同一个 package 下面就可以
> * private 方法（有异议），我的看法是私有方法如果逻辑很多，应该重构出来提供 public 方法或者新的 Class 进行重构；如果逻辑不多仍然保证不了无 bug ，可以使用反射来测试。其实 private 方法的测试是需要通过对 public 方法的测试来完成的，因为 private 方法总是会被 public 方法调用的。还有一种测试方法就是放宽访问限制，private 方法改为 protected，并且用 guava 的 @VisibleForTesting 注解标注放宽权限的方法。

### **13 如何设计适合测试的接口**

> * 1.Dependency Injection

如果把一种依赖写死在方法里肯定不利于测试，如果该依赖是一种强引用第三方服务的 sdk 你就痛苦了，如配置类初始化时需要连接 zk 且无法注入

> * 2.Abstraction

包括类的抽象、方法的提取，代码越精简，测试越方便、越快速、越容易暴露问题

> * 3.开闭原则

面向扩展开发，面向修改闭合，不对老代码入侵，避免 UT 重复修改

> * 4.慎重声明 static 方法

最好的 static 方法是完全不依赖任何第三方服务自己可以实现业务逻辑的代码，如果依赖第三方，使用 reference 传入而不是写死在 class 或 method 里

> * 5.测试类而不是实现

单元测试测试的对象是类，测试类的功能在各种情况下是否符合预期，而不是测试实现。所以我们只需要测试能够跟其他类交互的 public 方法就可以了。这样的一个好处就是，如果哪天需要重构代码的实现，或者换一个算法实现某些方法，但功能不变的情况下，UT 是可以复用的。如果针对实现来测试，如果哪天要重构代码实现，那 UT 就会 fail 掉。

> * ……待续……

## 测试覆盖率

一个仁者见仁智者见智的问题，不做深入讨论了。

个人建议工具方法（保证正确性以及边界条件不出错）、核心流程（复杂的条件判断尤其需要 UT 保证）需要重点覆盖，底层接口如 DAO、简单的 Service 封装可以不用写。

## IDEA Plugin

推荐一个 JUnitGeneratorV2.0，可以通过 Command+N 来生成 Test 类或者直接在类名上使用 alt+enter 来生成。

## 总结

没有测不了的代码，只有测试不够方便的代码，大多源于设计不够合理。

写单元测试的难易程度跟代码的质量关系最大，并且是决定性的。项目里无论用了哪个测试框架都不能解决代码本身难以测试的问题，所以如果你遇到的是“我的代码里依赖的东西太多了所以写不出来单测”这样的问题的话，需要去看的是如何设计和重构代码，而不是这篇文章。

## Reference

> * [https://dzone.com/articles/7-popular-unit-test-naming](https://dzone.com/articles/7-popular-unit-test-naming) \( [https://dzone.com/articles/7-popular-unit-test-naming](https://dzone.com/articles/7-popular-unit-test-naming) \)
> * [https://github.com/mockito/mockito/wiki/What%27s-new-in-Mockito-2](https://github.com/mockito/mockito/wiki/What%27s-new-in-Mockito-2) \( [https://github.com/mockito/mockito/wiki/What%27s-new-in-Mockito-2](https://github.com/mockito/mockito/wiki/What%27s-new-in-Mockito-2) \)

