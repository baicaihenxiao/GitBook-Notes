# 最全 | Redis可视化工具横向评测

[https://mp.weixin.qq.com/s/pV8mCq2bIyThH9e4Drlu-g](https://mp.weixin.qq.com/s/pV8mCq2bIyThH9e4Drlu-g)

来源：suo.im/66KSqr

### 1 命令行

不知道大家在日常操作redis时用什么可视化工具呢？

以前总觉得没有什么太好的可视化工具，于是问了一个业内朋友。对方回：你还用可视化工具？直接命令行呀，redis提供了这么多命令，操作起来行云流水。用可视化工具觉得很low。

命令行的鄙视用工具的，用高端工具的鄙视低端工具的，鄙视链一直存在。虽然用命令行自己也可以，但是总感觉效率上不如用工具，在视觉上不那么直观。尤其是看json的时候，在命令行就很不友好。

大佬朋友说：谁说命令行就不能格式化json了？可以利用iredis，用`|`将redis通过pipe用shell的其他工具，比如`jq/fx/rg/sort/uniq/cut/sed/awk`等处理。还能自动补全，高亮显示，功能很多

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-180029.jpg)

好吧 ，确实牛逼。附上这个工具的官网地址，喜欢用命令行的朋友可以去试一试，绝对能让喜欢命令行的你爽的飞起来。

> [https://iredis.io/](https://iredis.io/)

但是我相信大多数开发者还是习惯用可视化工具。我自己也用过不少redis的可视化工具。今天就细数下市面上流行的各个可视化的工具的优劣势。帮助你找到最好的redis可视化工具。提升debug效率。

如果你想直接看最终总结，可以直接拉到文章的末尾。

### 2 可视化工具分类

按照redis可视化工具的部署来分，可以分成3大类

* 桌面客户端版
* web版
* IDE工具的plugin

桌面版这次评测的软件如下：

* redis desktop manager
* medis
* AnotherRedisDesktopManager
* fastoredis
* redis-plus
* red

Web版本评测的软件如下：

* redis-insight

IDE插件版本，这里只评测IntelliJ IDEA的插件，eclipse的就不作介绍了

* Iedis2

### 3 redis desktop manager

这个工具应该是现在使用率最广的可视化工具了。存在时间很久。经过了数次迭代。跨平台支持。以前是免费的，现在为收费工具。试用可以有半个月的时间。链接为：

> [https://redisdesktop.com/](https://redisdesktop.com/)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180029734-180029.jpg)

**评测：**

之前用觉得功能还行，就是界面UI丑了点。最近下了最新版，感觉经过了那么长时间迭代，界面看着也还凑合。该有的功能都有。界面看着比较简洁，功能很全。

key的显示可以支持按冒号分割的键名空间，除了基本的五大数据类型之外，还支持redis 5.0新出的Stream数据类型。在value的显示方面。支持多达9种的数据显示方式。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180029831-180029.jpg)

命令行模式也同以前有了很大的进步，支持了命令自动提示。

![img](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-M5LMBM-KNwLIye8nLEI%2Fuploads%2FlIo3wnM2WTZVJdwdfvit%2Ffile.gif?alt=media)

从功能看上去中规中矩，使用起来便捷。最大的缺点就是不免费。个人使用的话，大概一年要200多RMB的价格。

###

### 4.medis

现阶段我使用率最高的redis可视化工具。界面符合个人审美。布局简洁。跨平台支持，关键是免费。链接为：

> [http://getmedis.com/](http://getmedis.com/)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180029927-180030.jpg)

**评测：**

颜值挺高，功能符合日常使用要求。对key有颜色鲜明的图标标识。在key的搜索上挺方便的，可以模糊搜索出匹配的key，渐进式的scan，无明显卡顿。在搜索的体验上还是比较出色的。

缺点是不支持key的命名空间展示，不支持redis 5.0的stream数据类型，命令行比较单一，不支持自动匹配和提示。支持的value的展现方式也只有3种

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180030021-180030.jpg)

###

### 5.AnotherRedisDesktopManager

一款比较稳定简洁的redis UI工具。链接为：

> [https://github.com/qishibo/AnotherRedisDesktopManager](https://github.com/qishibo/AnotherRedisDesktopManager)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180030117-180030.jpg)

**评测：**

很中规中矩的一款免费的redis可视化工具，基本的功能都有。有监控统计，支持暗黑主题，还支持集群的添加。

缺点是没什么亮点，UI很简单，不支持stream数据类型。命令行模式也比较单一。value展示支持的类型也只有3种。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180030213-180030.jpg)

###

### 6.FastoRedis

FastoRedis之前没听到过。然后去下了体验了下。

使用这款工具首先得去官网注册账号。这款软件是收费软件，虽然跨平台，但是试用只有一天的时间。链接为：

> [https://fastoredis.com/](https://fastoredis.com/)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180030310-180030.jpg)

**评测：**

毕竟是收费软件，虽然界面一股浓浓的windows风格，乍看上去有点像redis desktop manager，但是就功能而言。确实不错，支持了集群模式和哨兵模式，key的命名空间展示，redis 5.0的stream数据类型也支持。

命令行模式支持自动提示补全

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180030407-180030.jpg)

value的显示支持树状，表格状等等显示方式。令我惊讶的是，值对象支持多达17种渲染方式，

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180030538-180030.jpg)

总的来说，除了界面UI交互略生硬，还有是一款收费软件之外，还是一款很不错的redis可视化工具。

###

### 7.RedisPlus

一款开源的免费桌面客户端软件链接：

> [https://gitee.com/MaxBill/RedisPlus](https://gitee.com/MaxBill/RedisPlus)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180030644-180030.jpg)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180030743-180030.jpg)

**评测：**

没什么亮点，也就基本功能。加分项可能也就是有一个监控。其他的都很普通 。甚至于这款软件连命令行模式都没有。用的是javafx开发，按道理说，应该是跨平台的软件 ，但是提供的下载地址，并没有mac的直接安装包。况且就算是跨平台的吧。

###

### 8.Red

这是一款在苹果app store下载的redis可视化工具，免费链接：

> Mac用户可以去app store里面搜

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180030899-180031.jpg)

**评测：**

只支持Mac端，颜值还是不错的。功能中规中矩。基本功能都有，支持key命名空间的展示。

###

### 9.Redis Insight

这个软件来头挺大的，是redis labs出的一款监控分析级别的redis可视化工具。这款软件是web版的

那redis labs是啥公司，redis labs创立于2011年，公司致力于为Redis、Memcached等流行的NoSQL开源数据库提供云托管服务。可以算是专门致力于redis云的一家专业公司。他们的提供的软件中，除了可以连接企业私有的redis服务，也可以连接他们的redis云。链接：

> [https://redislabs.com/redisinsight/](https://redislabs.com/redisinsight/)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180030996-180031.jpg)

**评测：**

虽然是web版本，但是这个软件超越了我对redis可视化工具的认识，一看界面就觉得很专业，不像是个人开发出来的开源产品。我发现key的查询和浏览只是这里的一个功能模块而已

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180031091-180031.jpg)

命令行方面：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180031190-180031.jpg)

除了有命令补全提示，右边还有相关命令的文档解释。怎么样，是不是超人性化呢？

同样支持redis 5.0的Stream数据类型

下面的三个功能，是需要在server端安装他们家的其他redis模块的。分别是可查询的图表，redis的时间序列展示和全文本查询功能。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180031286-180031.jpg)

最牛逼的是，redisInsight竟然还支持rdb的分析功能，之前分析rdb的存储分布，有点经验的都会用rdb-tools去分析。而redisInsight竟然把这个都集成进去了。我之前用这个分析了公司生产环境的rdb，找出了导致数据量增长过快的原因，简直是一个神器。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180031384-180031.jpg)

这是我上次利用这个软件分析rdb出来的结果。很明确的找到了哪个key占据内存过大。

在分析功能中的Profiler能监听一段时间内所有执行的redis命令 ，Slowlog能显示出执行比较慢的redis命令。

除此之外，这个软件还能批量操作

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180031484-180031.jpg)

RedisInsight这个可视化工具对redis的覆盖之全面令人咋舌。虽然他的查询key的功能算不上优秀，但是他的全面性和分析监控方面，确实是其他redis可视化工具难以企及的，况且颜值还那么高，强烈推荐。

###

### 10.Iedis2

Iedis是一款基于IntelliJ IDEA的插件，在IDEA的plugin市场里就可以搜到，但是为收费插件。可试用7天

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180031633-180031.jpg)

**评测：**

作为IDEA的插件，当然是跨平台的，风格完全遵从于IDEA，颜值有保障。从功能上来说，Iedis也是不含糊。基本查询功能基本上挑不出毛病。加上IDEA的使用习惯，让你用起来得心应手，不需要另外打开软件。在代码和插件窗口中切换也是能提高效率的

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180031865-180032.jpg)

这个插件最大的特点就是能支持lua脚本的编写和调试，这在其他软件中是不曾看到的。以前在一个业务中大量用了lua进行redis操作，虽然尝到了redis lua原子性和性能上的甜头，但是在编写调试的时候，那叫一个痛苦，因为不能在debug所以每次都需要返回一个值来检查是哪里出了错。看到这个工具，悔恨没早点发现这个插件，付费也愿意

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180032015-180032.jpg)

这个插件还能支持慢命令的查看

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180032183-180032.jpg)

总的来说，Iedis除了需要付费，其他的一切都看上去很美好。价格是。。。$139/年。还是美元，看到这个价格，是不是长叹一口气呢。

###

### 11.总结

对于前面介绍的8款redis可视化工具，我总结了一个表格，供大家参考和比较

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/02/640-20200802180032281-180032.jpg)

相信看到这里，你心里一定有答案了。好的工具能让你事半功倍，从而节约大量的时间和成本，希望大家在日常开发中，能挑选好的工具，以最快的效率解决最复杂的事情。
