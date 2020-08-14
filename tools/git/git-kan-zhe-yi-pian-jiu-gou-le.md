# Git 看这一篇就够了

{% embed url="https://mp.weixin.qq.com/s/H6nNvvfvrPMJtnMV0JhsMg" %}





今天讲下 Git 的实现原理，知其所以然才能知其然；梳理了日常最常用的 12 个命令，分为三大类分享给你。

本文的结构如下：

1. **作者和开发原由**
2. **Git 的数据模型**
3. **常用命令**
4. **资源推荐**

### 作者和开发原由

> Talk is cheap. Show me the code.

这句话就出自 Linux 和 Git 的作者 `Linus Torvalds`。

原本 Linux 内核的版本控制系统是用的 BitKeeper，然而 2005 年，BitMover 公司不再让 Linux 开发团队免费使用了。。

Linus 一听，不给用了？老子自己写！

于是，大佬十天之内完成了 Git 的第一个版本。

所以 Git 是一个免费的、开源的版本控制系统。

#### 版本控制系统

版本控制其实每个人都用过，那些年修改过的简历：

> 小齐简历 2012 版 小齐简历 2013 版 小齐简历 2014 版 小齐简历 2015 版 小齐简历 2016 版 小齐简历 2017 版 小齐简历 2018 版 小齐简历 2019 版 ...

还有那些年打死都不再改的毕业论文：

> 毕业论文最终版 毕业论文最最终版 毕业论文最最最终版 毕业论文最最最最终版 毕业论文最终不改版 毕业论文最终真不改版 毕业论文最终真真不改版 毕业论文最终打死不改版 毕业论文最终打死不改版 2 ...

没错，这就是本地版本控制系统。

很明显，好处是简单，但是只能一个人在这改，无法和他人完成合作。那么以下两种主流的版本控制系统应运而生。

**1. 集中化版本控制系统**

Centralized Version Control Systems \(CVCS\)

比如：CVS, Subversion, Perforce, etc.

这种版本控制系统有一个单一的集中管理的服务器，保存所有文件的最新版本，大家可以通过连接到这台服务器上来获取或者提交文件。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/01/640-20200701194335591.jpg)

这种模式相对本地版本控制系统是有所改进的，但是缺点也很明显，如果服务器宕机，那么轻则耽误工作、重则数据丢失。于是分布式版本控制系统应运而生。

**2. 分布式版本控制系统**

Distributed Version Control Systems \(DVCS\)

比如：Git, Mercurial, Bazaar, etc.

分布式的版本控制系统会把**代码仓库完整地镜像**下来，这样任何一个服务器发生故障，都可以用其他的仓库来修复。

更进一步，这种模式可以更方便的和不同公司的人进行同一项目的开发，因为两个远程代码仓库可以交互，这在之前的集中式系统中是无法做到的。

**那么什么叫“把代码仓库完整地镜像下来”呢？**

CVCS 每个版本存放的是当前版本与前一个版本的差异，因此也被称作基于差异的版本控制 \(delta-based\)；

Git 存储的是所有文件的一个快照 \(snapshot\)，如果有的文件没有修改，那就只保留一个 reference 指向之前存储的文件。

不是很好理解？那接着看吧～

### Git 的数据模型

**1. 什么是快照 \(snapshot\) 呢？**

首先我们来学两个 Git 中的术语：

* blob, 就是单个的文件；
* tree, 就是一个文件夹。

快照则是被追踪的最顶层的树。

比如我的“公众号”文件夹的这么一个结构：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/01/640-20200701194335703.jpg)

那么一个快照就是追踪的“公众号”这颗树。

**2. 本地库的数据模型**

Git 记录了每个快照的 parent，也就是当前这个文件夹的上一个版本。

那么快照的迭代更新的过程就可以表示为一个**有向无环图**，是不是很熟悉？我们在「拓扑」那篇文章里讲过，忘了的小伙伴快去公众号内回复「拓扑」获取拓扑的入门文章吧～

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/01/640-20200701194335756.jpg)

每个快照其实都对应了一次 `commit`，我们用代码来表示一下：

```text
class commit {
  array<commit> parents
  String author
  String message
  Tree snapshot
}
```

这就是 Git 的数据模型。

blob, tree, snapshot 其实都一样，它们在 Git 中都是`对象`，都可以被引用或者被搜索，会基于它们的 `SHA-1 hash` 进行寻址。

> `git cat-file -t`: 查看每个 SHA-1 的类型; `git cat-file -p`: 查看每个对象的内容和简单的数据结构。

但是通过这个哈希值来搜索也太不方便了，毕竟这是一串 40 位的十六进制字符，就是第二部分 `git log` 里输出的那个`编码`。

因此，Git 还给了一个引用 `reference`。

比如，我们常见的 `HEAD` 就是一个特殊的引用。

本地库就是由 `对象` 和 `引用` 构成的，或者叫 `Repositories`.

在硬盘上，Git 只存储 `对象` 和 `引用`，所有的 Git 命令都对应提交一个快照。

那有哪些常用命令呢？

### 常用命令

本章分三大部分介绍日常常用命令：

* 本地操作
* 和远程库的交互
* 团队协作 - 分支

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/01/640-20200701194335829.jpg)

#### 本地操作

在学习常用命令之前，你首先需要知道的 Git 的「三个分区」和对应的文件的「三种状态」：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/01/640-20200701194335928.jpg)

* `工作区`：就是你本地实际写代码的地方，无论你是用 vim 直接改也好，还是在 IDE 里写，都无所谓。
* * 对应的文件状态是：`modified`，已修改，但还没保存到数据库中。
* `暂存区`：就是临时存放的地方。
* * 对应的文件状态是：`staged`，Git 已经对该文件做了标记，下次提交知道要包含它。
* `本地库`：存放本地历史版本信息。
* * 对应的文件状态是：`committed`，文件已经安全的保存在本地数据库中。

**1. $ git add**

工作区改完了代码，就用 `git add` 提交到暂存区。

这里如果文件改动的比较多，但又不是每个都需要提交，我会设置 `git ignore file`，就表示这些文件不要提交，比如在 build project 的时候会自动生成的那些文件等等。

**2. $ git commit -m "comment"**

从暂存区提交到本地库，就需要用 commit。

一般后面都会跟个 `-m` 加句 `comment`，简单说下改动的内容或者原因，我们公司大家默认也会把 `Jira`链接附上，这样就知道这个改动对应哪个任务。

那如果想再改，再重新 `git add` 即可，但是 `commit` 这句需要改成

```text
$ git commit --amend
```

这样就还是一条 git log 信息。

**3. $ git log**

`git log` 可以查看到提交过的信息，从近到远显示每次 commit 的 comment 还有作者、日期等信息，比如大概长这个样子：

```text
commit 5abcd17dggs9s0a7a91nfsagd8ay76875afs7d6
Author: Xiaoqi<xiaoqi@163.com>
Date: xxx xxx xxx
改了 Test 文件
```

commit 后面的这个`编号`，是每次历史记录的一个`索引`。比如如果需要对版本进行前进或者后退的时候，就需要用到它。

这样打印的 log 太多，更简洁的打印方式是：

```text
$ git log --oneline
```

就一行打印出来了。

或者：

```text
$ git reflog
```

更常用一些。

**4. $ git reset**

那我们刚刚说过，如果需要前进或退回到某个版本，就用

```text
$ git reset --hard <编号>
```

这样就直接跳到了这个`编号`对应的那个版本。

那么这个 `hard` 是什么意思呢？

这里有 3 个参数：`hard`, `soft`, `mixed`，我们一一来说一下。

回到我们最重要的这张图上来：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/01/640-20200701194335928.jpg)

我们刚刚说的前进或后退到某一版本，是对`本地库`进行的操作。

那有个问题： **本地库的代码跳到那个版本之后，工作区和暂存区的代码就和本地库的不同步了呀！**

那这些参数就是用来控制这些是否**同步**的。

**$ git reset --hard xxx**

三个区都同步，都跳到这个 xxx 的版本上。

**$ git reset --soft xxx**

前面两个区不同步，就只有本地库跳到这个版本。

**$ git reset --mixed xxx**

暂存区同步，工作区不动。

所以呢，用的多的就是 hard.

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/01/640-20200701194335829.jpg)

#### 远程交互

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/01/640-20200701194336187.jpg)

和远程库的交互主要是`推`、`拉`，也就是写入和读取。

**5. $ git push**

小齐写完了代码，要提交到公司的代码库里，这个过程要用 `git push`.

当然了，这么用会被打的。。毕竟还要 cr 呢。

**5. $ git clone**

新来的实习生首先要 clone 整个项目到本地来，然后才能增删改查。

当然了实际工作中也没人这么用。。因为每家公司都会有自己包装的工具。不过如果是做 Github 上的开源项目，就用得上了。

**6. $ git pull**

小齐提交了新的代码之后，领导要审查呀，所以用 `git pull` 把最新的代码拉取下来瞅瞅。

实际上呢，

```text
git pull = fetch + merge
```

**7. $ git fetch**

`git fetch` 这个操作是将远程库的数据下载到本地库，但是工作区中的文件没有更新。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/01/640-20200701194336396.jpg)

而要谈 `get merge`，我们还需要先讲下`分支`。

`merge` 是 `git pull` 默认的选项，合并其实还有另外一种方法：`rebase`，中文叫做**变基**。

**8. $ git rebase**

rebase 的作用更多的是来整合分叉的历史，可以将某个分支上的所有修改都移到另一分支上，就像是变了基底。

#### 分支与合并

首先我们来看几个关于分支的基本操作：

**9. 查看分支：**

```text
$ git branch
```

类似于`ls`，能够列出当前所有分支。

`git branch -v` 能够显示更多信息。

**10. 创建分支：**

```text
$ git branch <branchName>
```

**11. 切换分支：**

```text
$ git checkout <branchName>
```

有了分支之后必然会有合并：

**12. 合并分支：**

```text
$ git merge <branchName>
```

而合并时就可能会有冲突，什么时候会有冲突呢？：

**在同一个文件的同一个位置修改时。**

因为 Git 会努力的把你们改动不同的地方合并在一起，但如果实在是在同一个地方改的，那它也没办法了，只能留给程序员去手动处理了。

当然了，每个命令延伸下去还有无限多个，本文不可能涵盖全部，所以在此重磅推荐齐姐精心挑选的三大学习资源，大家可以自行享用～

### 学习资源

#### git help

其实我个人使用最多的是`git help`

真心方便又好用啊！

比如 `git help pull`:

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/01/640-20200701194336649.jpg)

先介绍了有哪些参数，然后 description 详细解释了它的工作原理，下面还有图解，有木有太香！！

不过这种方式更像是 `cheatsheet`，当你已经知道了这个命令、只是忘了它的用法的时候去查。

如果你想系统的学习，那么下面 👇 的更适合你。

#### Pro Git

这本书是强烈推荐了！！

Pro Git 这本书不仅讲了 Git 的基础用法、高级用法，以及最后还深入讲解了 Git 的原理，非常细致全面。

书的电子版也能在网站上直接下载。

英文版：

* `https://git-scm.com/book/en/v2`

中文版：

* `https://git-scm.com/book/zh/v2`

#### 玩游戏

Practice makes perfect!

推荐一个宝藏资源：玩游戏来练 Git

项目：`https://github.com/pcottle/learnGitBranching`

网址：`https://learngitbranching.js.org/`

我熟悉很多工具都是通过小游戏来练习的，比如 vim 的操作，还是蛮推荐这种方式的。就不剧透啦，大家自己去探索吧～

