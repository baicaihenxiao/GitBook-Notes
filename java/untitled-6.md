# 科普 \| Shell中傻傻分不清楚的TOP3



[https://mp.weixin.qq.com/s?\_\_biz=MzI4ODQ3NjE2OA==&mid=2247487114&idx=1&sn=a2eb1dcaf15bc72191f649ced1d74b5e&chksm=ec3c93eddb4b1afb8ab6772725418986684c62e8ea04e9470eca08f9ff4a75ae9952499aa8d5&mpshare=1&scene=1&srcid=&sharer\_sharetime=1592407791350&sharer\_shareid=393f249533d421d13c2402bd43e74356&key=e7bdc15d285ac0a9c6a26cbe6250f4e980268140c63fe3057c128016f0d5b896d5c3723df4f61d350b8462b5d62012f2d31d39871d7700a5d9856390e65296c6f1c8a813049683319536c327254c5b84&ascene=1&uin=MjY3NTU5NjEwMg%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=en&exportkey=Ad444lyZ%2FuPGK8nEzc31PcQ%3D&pass\_ticket=0LrZakFu76KCYOhpLKD%2BCxbND%2BEak%2BD%2FjBb%2FaEybhQCiuVMsB0giTEhqidzglEA9](https://mp.weixin.qq.com/s?__biz=MzI4ODQ3NjE2OA==&mid=2247487114&idx=1&sn=a2eb1dcaf15bc72191f649ced1d74b5e&chksm=ec3c93eddb4b1afb8ab6772725418986684c62e8ea04e9470eca08f9ff4a75ae9952499aa8d5&mpshare=1&scene=1&srcid=&sharer_sharetime=1592407791350&sharer_shareid=393f249533d421d13c2402bd43e74356&key=e7bdc15d285ac0a9c6a26cbe6250f4e980268140c63fe3057c128016f0d5b896d5c3723df4f61d350b8462b5d62012f2d31d39871d7700a5d9856390e65296c6f1c8a813049683319536c327254c5b84&ascene=1&uin=MjY3NTU5NjEwMg%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=en&exportkey=Ad444lyZ%2FuPGK8nEzc31PcQ%3D&pass_ticket=0LrZakFu76KCYOhpLKD%2BCxbND%2BEak%2BD%2FjBb%2FaEybhQCiuVMsB0giTEhqidzglEA9)





近来小姐姐又犯憨憨错误，问组内小伙伴`export`命令不会持久化环境变量吗？反正我是问出口了。。然后小伙伴就甩给了我一个《The Linux Command Line》PDF链接。感谢老大不杀之恩～

Shell是命令解释器，它会接受用户输入的各种命令，并传递给操作系统执行。它的作用类似于Windows系统的命令行。在UNIX或Linux系统中，Shell即是用户交互的界面，也是控制系统的脚本语言。当然现在用户也可以选择图形化界面做一些和操作系统的交互。层次示意图如下：![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9HUPyzUmGvqXDIzNiaabCkwSpdLDG2ibUZAelINENZfAMkI046P210cAA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于初学者来说，可能搞不清楚Shell怎么会有那么多分类，Shell的语法怎么那么随便...

小姐姐结合自己初学Shell傻傻分不清的问题点，主要从`Shell的种类`,`变量的分类`,`条件测试的表达`三个部分来介绍。

### Shell的种类

shell程序有**sh**，**bash**,**zsh**等分类，我从网上找到一张图可以看出shell程序的发展史。![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY98Z6paJyH0cnV5MNqk9DXYia3x9s94L01LOrNhhbObnm4LJiauPcLpxTw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于这些Shell程序，其语法或多或少有一些差异，不过我们通常使用的都是bash。  


**Shell程序信息**

在Linux系统我们可以通过一些命令查看或修改当前Shell程序信息。![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9ibibPXOB8qtiaNGUQodVnhI4Clk7AiaiaxxGIj9RagrKm5YhfG0065cydRg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

一般发行版的Linux系统中，默认的shell程序就是bash。我们在写shell脚本时，通常也会在脚本文件头部指定bash作为脚本解释器。![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9MUZAsvLCHH43rG6ojhibPmibzODSOQk3GsWA2eYSUJwvLBc7kicyRzNbg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里多说一句，zsh有时也作为猿媛们的默认shell。zsh语法大多是和bash匹配的，也不会影响shell脚本的执行（因为脚本头部指定bash就还是bash：），也不会影响像小姐姐这样的渣渣使用。用它是因为它有神奇的开源框架 Oh My God.. 哦不，是 **Oh My Zsh** !!!

后面的内容我们还是以Linux系统中的bash为例来介绍：）

### 变量的分类

Shell是一门动态类型语言和弱类型语言，我们可以把变量理解为KV对，key是变量名，value是变量值。变量大体可以分为`环境变量`，`系统变量`，`用户定义的变量`三类。

**环境变量**

比如我们经常配置的`JAVA_HOME`就属于环境变量，这些变量是所有Shell程序运行时都可以使用的变量。关于环境变量的操作命令举例如下：![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9PDGxulEDar9M4IU74D1wM9vthOKVCL804nx4ETcwOW0gdEkDGu7egQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9ibdxKRGDiaO6bMv1uvfQCjXeicSMhzKPLNDn7F5LMP1uT9BCCxPibDOVibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

使用`export`命令定义的环境变量只在当前运行的shell进程中有效，结束进程就没了。所以我们要将配置变量定义在令小姐姐懵逼的一系列配置文件中，持久化下来。

说起配置文件，又不得不先提下shell程序和用户的Interactive和Login模式:\)

* **Interactive & Non-Interactive**

`Interactive`通常是指读入写出数据都是从用户的terminal，也就是我们平时用命令行打开终端就是Interactive模式，而执行一个shell脚本就是`Non-interactive`模式。怎么检验当前shell运行的模式是不是Interactive呢？小姐姐从GNU网站拷贝了一段脚本：

```text
case "$-" in
*i*)	echo This shell is interactive ;;
*)	echo This shell is not interactive ;;
esac
```

结果如上所述。

* **Login & Non-Login**

`Login模式`指的是用户成功登录后开启的shell进程，这时候会读取`/etc/passwd`下用户所属的shell去执行。![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9lneQiceZPmMia5bCoibbFXGoz9VOQJyx5ib9ia1mvmPlUZickTfIK9dIY5Dg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`Non-login模式`指的是非登录用户状态下开启的shell进程，我们可以通过`echo $0`区分。![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9PoibMicBicaVEHm5GUiabxJUicwK0VW7e5ViaicoTTP8yxRqIo9nKClbNfpwg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

扯这么多是因为配置文件的加载顺序和shell进程是否运行在Interactive和Login模式有关系:D

* **配置文件加载顺序**

![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9zcUFmkrYZj4iavlA1K9nyKOZ8Bca6ar5nP5oqjyVrEtLeeWL9UzdIyA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这是阿姨从网上粘的图。bash支持的配置文件有/etc/profile,~/.bashrc等。

当调用一个**Interactive&Login**模式的shell进程时，配置文件的加载顺序为：

`/etc/profile` —&gt;`( ~/.bash_profile, ~/.bash_login, ~/.profile)其中之一` —&gt;`~/.bash_loginout`\(退出shell时调用\)

当调用一个**Interactive&non-Login**模式的shell进程时，配置文件的加载顺序为：

`/etc/bash.bashrc` —&gt;`~/.bashrc`

当调用一个`non-nteractive`模式的shell进程时，通常是执行脚本时，此时配置项是从环境变量中读取和执行的，也就是`env`命令输出的配置项。

另外，在开启一个shell进程中，有一些参数的值也会影响到配置文件的加载。如--rcfile，--norc等。这些参数的含义值可以使用`man bash`进一步了解。只要保持默认值，其实就是我们上面介绍的配置文件加载顺序。

还有，在发行版的Linux系统中，Interactive&Login模式下的~/.bash\_profile, ~/.bash\_login， ~/.profile并不一定是三选一，看一下这三个脚本的内容会发现他们会继续调用下一个它想调用的配置文件，这样就可以避免配置项可能需要在不同的配置文件多次配置的弊端了。如centos7.2 中 ~/.bash\_profile文件中实际调用了 ~/.bashrc文件。![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9iabiaDdgn24aicE3JHAd3z3usfUUH1b8K9GAfa5QibticmMKjjQk2jiamOXg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

按照模式和参数设置启动的shell程序的配置文件加载流程图如下：![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9YffnBde6h3ibJhFKFxsBll15K03AeJUWz8HjlNaR8x7ib19mExjIwSTA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

好了，到目前我们总算把环境变量中配置文件的加载顺序理清了。下面列举一些常用的Shell环境变量吧。![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY96l2wL9YegNnZ2RXEqC9C1cFc2bp6Ne6DClYqPdiaoqHIhVSNia4YDYdA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**系统变量**

Shell中系统变量主要在对**参数判断和命令返回值判断**时使用，包括脚本和函数的参数和返回值判断。没啥可说的，主要难记且脚本中经常出现：![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9yQzAjrRU742s2t6NAoL8DYAaD2OWOSk66CjibXfsQ9tfKKab5D4fGRA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**用户自定义的变量**

是指我们在使用命令或脚本时定义的变量，因为shell是弱类型语言且语法XX，这里主要谈谈初学时的几个坑爹点：

* **“=” 左右两边不能有空格**

![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9s5BCNHbfzdsQ16hYKAhiaEpte80aKoZGLUzn5qlSKU2hpS1cJ24NrlQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)你懂我说的意思了吧。。

* **引用**

所谓引用，指的是将字符串用引用符号包括起来，以防止其中的特殊符号被Shell解释为其他涵义。

常用的引用符号如下:![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9ODOeQODoVadicp9JnoRAXsYtjs3NCbCQZL4mgHhdkZIE1BibpmYNkhicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

* **$**

前面我们其实一直在用“$变量名来表示某个变量名的值，这其实也正是$的作用。

* **Shell中变量名的大小写是敏感的**

好了就这么多吧。

### 条件测试的表达

Shell脚本中除了变量，还经常出现的语法就是条件测试的判断。不会写脚本的开发小姐姐不是好运维，我们来一起侃侃吧。

**基本语法**

在Shell程序中，当指定的条件为真时，整个条件测试的返回值为 0；反之，如果指定的条件为假时，整个条件测试的返回值为 非0。

条件测试表达式的书写有`test expression` 和 `[ expression ]`两种形式，注意后者的空格一定不能省！！

脚本中经常出现的有字符串测试、数字测试、文件测试、逻辑操作符测试。我们一起看下：）

**字符串测试**

![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9ZbAUUv2OXcDh16RV7ibBLHmQ1ibTOYbUUB7hBHzpk8e9bRsd9o5DIwYA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 注意：这里运算符 左右两边又一定要有空格了（下同），这样shell才能将之当成命令执行。

**数字测试**

![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9XTSeqdDNyDKzpetyckianBGzCXH87873KhSAz3Sv8AKyUYmzrlAW58A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**文件测试**  


![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9yicHHqx7Crzrfj8oUqEuYrs7fgiay283cQ3ZUl1f4hD45icCRDDeibFZvg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**逻辑操作符测试**

![](https://mmbiz.qpic.cn/mmbiz_png/f93EtXu3ZkicRhAdmf1rDibY0fynw3NnY9aYm4Z7qAeRtNdp2fOIWqIfIs7NGvDmhUVwxyaqGNJBticiaAsTXlN5Gg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`收藏 在看 转发` 起来，小姐姐就算你条件测试过关了&gt;\_&lt;

参考资料：

\[1\].《Shell从入门到精通》

\[2\].https://www.edureka.co/blog/types-of-shells-in-linux/

\[3\].http://www.penguintutor.com/linux/basic-shell-reference

\[4\].https://apple.stackexchange.com/questions/361870/what-are-the-practical-differences-between-bash-and-zsh

\[5\].https://sunlightmedia.org/bash-vs-zsh/

\[6\].https://unix.stackexchange.com/questions/439042/debian-read-order-of-bash-session-configuration-files-inconsistent

\[7\].https://www.gnu.org/software/bash/manual/html\_node/Bash-Startup-Files.html

\[8\].http://howtolamp.com/articles/difference-between-login-and-non-login-shell/

\[9\].https://shreevatsa.wordpress.com/2008/03/30/zshbash-startup-files-loading-order-bashrc-zshrc-etc/

