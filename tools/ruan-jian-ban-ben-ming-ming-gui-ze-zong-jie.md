# 软件版本命名规则总结

[https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247506069\&idx=1\&sn=ec2245ee099d9040c6db36880852f545](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247506069\&idx=1\&sn=ec2245ee099d9040c6db36880852f545)





来源 |  cnblogs.com/sdjxcolin/archive/2007/07/02/803376.html&#x20;

* [**01、GNU 风格的版本号命名格式 :**](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)
* [**02、Windows 风格的版本号命名格式 :**](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)
* [**03、.Net Framework 风格的版本号命名格式:**](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)
* [**04、应根据下面的约定使用这些部分：**](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)
* [**05、版本号管理策略**](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)
* [**06、GNU 风格的版本号管理策略：**](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)
* [**07、Window 下的版本号管理策略：**](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)
* [**08、正式版，不同类型的软件的正式版本通常也有区别。**](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)

***

版本控制比较普遍的 3 种命名格式 :

### [01、GNU 风格的版本号命名格式 :](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)

主版本号 . 子版本号 \[. 修正版本号 \[. 编译版本号 ]] Major\_Version\_Number.Minor\_Version\_Number\[.Revision\_Number\[.Build\_Number]] 示例 : 1.2.1, 2.0, 5.0.0 build-13124

### [02、Windows 风格的版本号命名格式 :](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)

主版本号 . 子版本号 \[ 修正版本号 \[. 编译版本号 ]] Major\_Version\_Number.Minor\_Version\_Number\[Revision\_Number\[.Build\_Number]] 示例: 1.21, 2.0

### [03、.Net Framework 风格的版本号命名格式:](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)

主版本号.子版本号\[.编译版本号\[.修正版本号]] Major\_Version\_Number.Minor\_Version\_Number\[.Build\_Number\[.Revision\_Number]][版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号由二至四个部分组成：主[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号、次[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号、内部[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号和修订号。主[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号和次[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号是必选的；内部[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号和修订号是可选的，但是如果定义了修订号部分，则内部[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号就是必选的。所有定义的部分都必须是大于或等于 0 的整数。

### [04、应根据下面的约定使用这些部分：](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)

Major ：具有相同名称但不同主版本号的程序集不可互换。例如，这适用于对产品的大量重写，这些重写使得无法实现向后兼容性。

Minor ：如果两个程序集的名称和主[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号相同，而次[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号不同，这指示显著增强，但照顾到了向后兼容性。例如，这适用于产品的修正版或完全向后兼容的新[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)。

Build ：内部版本号的不同表示对相同源所作的重新编译。这适合于更改处理器、平台或编译器的情况。

Revision ：名称、主[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号和次[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号都相同但修订号不同的程序集应是完全可互换的。这适用于修复以前发布的程序集中的安全漏洞。

程序集的只有内部版本号或修订号不同的后续版本被认为是先前版本的修补程序 (Hotfix) 更新。

### [05、版本号管理策略](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)

### [06、GNU 风格的版本号管理策略：](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)

1．项目初[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)时，[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号可以为 0.1 或 0.1.0, 也可以为 1.0 或 1.0.0，如果你为人很低调，我想你会选择那个主[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号为 0 的方式；2．当项目在进行了局部修改或 bug 修正时，主版本号和子版本号都不变，修正版本号加 1；3. 当项目在原有的基础上增加了部分功能时，主[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号不变，子[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号加 1，修正[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号复位为 0，因而可以被忽略掉；4．当项目在进行了重大修改或局部修正累积较多，而导致项目整体发生全局变化时，主版本号加 1；5．另外，编译[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号一般是编译器在编译过程中自动生成的，我们只定义其格式，并不进行人为控制。

### [07、Window 下的版本号管理策略：](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)

1．项目初版时，[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号为 1.0 或 1.00；2. 当项目在进行了局部修改或 bug 修正时，主版本号和子版本号都不变，修正版本号加 1；3. 当项目在原有的基础上增加了部分功能时，主[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号不变，子[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号加 1，修正[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号复位为 0，因而可以被忽略掉；4. 当项目在进行了重大修改或局部修正累积较多，而导致项目整体发生全局变化时，主版本号加 1；5. 另外 , 编译[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号一般是编译器在编译过程中自动生成的，我们只定义其格式，并不进行人为控制。

另外，还可以在版本号后面加入 Alpha、Beta、Gamma、Current、RC (Release Candidate)、Release、Stable 等后缀，在这些后缀后面还可以加入 1 位数字的版本号。

对于用户来说，如果某个软件的主[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号进行了升级，用户还想继续那个软件，则发行软件的公司一般要对用户收取升级费用；而如果子[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号或修正[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)号发生了升级，一般来说是免费的。

\=====附录软件版本名称=====

**α（alphal） 内部测试版**α版，此[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)表示该软件仅仅是一个初步完成品，通常只在软件开发者内部交流，也有很少一部分发布给专业测试人员。一般而言，该[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)软件的 bug 较多，普通用户最好不要安装。

**β（beta）外部测试版**该版本相对于α版已有了很大的改进，消除了严重的错误，但还是存在着一些缺陷，需要经过大规模的发布测试来进一步消除。这一版本通常由软件公司免费发布，用户可从相关的站点下载。通过一些专业爱好者的测试，将结果反馈给开发者，开发者们再进行有针对性的修改。该版本也不适合一般用户安装。

**γ（gamma）版**该[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)已经相当成熟了，与即将发行的正式版相差无几，如果用户实在等不及了，尽可以装上一试。

**trial（试用版）**试用版软件在最近的几年里颇为流行，主要是得益于互联网的迅速发展。该版本软件通常都有时间限制，过期之后用户如果希望继续使用，一般得交纳一定的费用进行注册或购买。有些试用版软件还在功能上做了一定的限制。

**unregistered（未注册版）**未注册版与试用版极其类似，只是未注册版通常没有时间限制，在功能上相对于正式版做了一定的限制，例如绝大多数网络电话软件的注册版和未注册版，两者之间在通话质量上有很大差距。还有些虽然在使用上与正式版毫无二致，但是动不动就会弹出一个恼人的消息框来提醒你注册，如看图软件acdsee、智能陈桥汉字输入软件等。

**demo 演示版**在非正式版软件中，该[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)的知名度最大。demo版仅仅集成了正式版中的几个功能，颇有点像 unregistered。不同的是，demo版一般不能通过升级或注册的方法变为正式版。

以上是软件正式版本推出之前的几个版本，α、β、γ可以称为测试版，大凡成熟软件总会有多个测试版，如 windows 98 的β版，前前后后将近有10个。这么多的测试版一方面为了最终产品尽可能地满足用户的需要，另一方面也尽量减少了软件中的bug 。而 trial 、unregistered 、demo有时统称为演示版，这一类版本的广告色彩较浓，颇有点先尝后买的味道，对于普通用户而言自然是可以免费尝鲜了。

### [08、正式版，不同类型的软件的正式版本通常也有区别。](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)

**release 最终释放版**该版本意味“最终释放版”，在出了一系列的测试版之后，终归会有一个正式版本，对于用户而言，购买该版本的软件绝对不会错。该版本有时也称为标准版。一般情况下，release不会以单词形式出现在软件封面上，取而代之的是符号 (r) ，如 windows nt(r) 4.0、ms-dos(r) 6.22 等。

**registered 注册版**很显然，该[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)是与 unregistered 相对的注册版。注册版、release和下面所讲的standard版一样，都是软件的正式[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)，只是注册版软件的前身有很大一部分是从网上下载的。

**standard 标准版**这是最常见的标准版，不论是什么软件，标准版一定存在。标准版中包含了该软件的基本组件及一些常用功能，可以满足一般用户的需求。其价格相对高一级版本而言还是“平易近人”的。

**deluxe 豪华版**顾名思义即为“豪华版”。豪华版通常是相对于标准版而言的，主要区别是多了几项功能，价格当然会高出一大块，不推荐一般用户购买。此[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)通常是为那些追求“完美”的专业用户所准备的。

**reference**该版本型号常见于百科全书中，比较有名的是微软的encarta系列。reference是最高级别，其包含的主题、图像、影片剪辑等相对于standard和deluxe版均有大幅增加，容量由一张光盘猛增至三张光盘，并且加入了很强的交互功能，当然价格也不菲。可以这么说，这一版本的百科全书才能算是真正的百科全书，也是发烧友们收藏的首选。

**professional（专业版）**专业版是针对某些特定的开发工具软件而言的。专业版中有许多内容是标准版中所没有的，这些内容对于一个专业的软件开发人员来说是极为重要的。如微软的visual foxpro标准版并不具备编译成可执行文件的功能，这对于一个完整的开发项目而言显然是无法忍受的，若客户机上没有foxpro将不能使用。如果用专业版就没有这个问题了。

**enterprise（企业版）**企业版是开发类软件中的极品（相当于百科全书中的reference版）。拥有一套这种[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)的软件可以毫无障碍地开发任何级别的应用软件。如著名的visual c++的企业版相对于专业版来说增加了几个附加的特性，如sql调试、扩展的存储过程向导、支持as/400对ole db的访问等。而这一[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)的价格也是普通用户无法接受的。如微软的visual studios 6.0 enterprise 中文版的价格为 23000 元。

**其他版本，除了以上介绍的一些版本外，还有一些专有版本名称。**

**update（升级版）**升级版的软件是不能独立使用的，该[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)的软件在安装过程中会搜索原有的正式版，如果不存在，则拒绝执行下一步。如microsoft office 2000升级版、windows 9x升级版等等。

**oem版**oem 版通常是捆绑在硬件中而不单独销售的版本。将自己的产品交给别的公司去卖，保留自己的著作权，双方互惠互利，一举两得。

**单机（网络）版**网络版在功能、结构上远比单机版复杂，如果留心一下软件的报价，你就会发现某些软件单机版和网络版的价格相差非常大，有些网络版甚至多一个客户端口就要加不少钱。

**普及版**该[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)有时也会被称为共享版，其特点是价格便宜（有些甚至完全免费）、功能单一、针对性强（当然也有占领市场、打击盗版等因素）。与试用版不同的是，该[版本](https://mp.weixin.qq.com/s?\_\_biz=MzU2NjIzNDk5NQ==\&mid=2247487217\&idx=1\&sn=a6428305479760448199d89eecc343f3\&scene=21#wechat\_redirect)的软件一般不会有时间上的限制。当然，如果用户想升级，最好还是去购买正式版。

* Enhance 增强版或者加强版 属于正式版
* Free 自由版
* Full version 完全版 属于正式版
* shareware 共享版
* Release 发行版 有时间限制
* Upgrade 升级版
* Retail 零售版
* Cardware 属共享软件的一种，只要给作者回复一封电邮或明信片即可。（有的作者并由此提供注册码等），目前这种形式已不多见。
* Plus 属增强版，不过这种大部分是在程序界面及多媒体功能上增强。
* Preview 预览版
* Corporation & Enterprise 企业版
* Standard 标准版
* Mini 迷你版也叫精简版只有最基本的功能
* Premium -- 贵价版
* Professional -- 专业版
* Express -- 特别版
* Deluxe -- 豪华版
* Regged -- 已注册版
* CN -- 简体中文版
* CHT -- 繁体中文版
* EN -- 英文版
* Multilanguage -- 多语言版
