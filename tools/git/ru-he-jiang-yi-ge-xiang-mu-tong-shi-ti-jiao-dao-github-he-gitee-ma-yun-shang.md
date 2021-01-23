# 如何将一个项目同时提交到GitHub和Gitee\(码云\)上



> [https://mp.weixin.qq.com/s/DyEu2wbEqkBduE0nup5ePA](https://mp.weixin.qq.com/s/DyEu2wbEqkBduE0nup5ePA)

如果你是GitHub的开源作者，是否因为GitHub访问慢或图片不显示而苦恼？你是否想让你的代码让更多人看到？那么，你可以将一套开源代码同时提交到多个开源平台。

当然，如果你已经在这么做了，但是只是手动的复制、分别上传，那么更本篇文章更值得你一看。

## 前言

GitHub几乎是每个程序员必逛的地方，但访问GitHub有一个明显的问题，就是网速比较慢，现在GitHub上很多图片信息还没办法正常显示。

Gitee（码云）这几年在国内发展势头迅猛，下面我们就以一套代码同时提交到GitHub和Gitee为示例来，来讲解如何配置Git达到同时上传代码到多个平台。

## GitHub上创建一个仓库

在GitHub上创建一个仓库：

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/01/23/640-190812.jpg)git

当然，如果对应的仓库已经存在，则可跳过此步骤。笔者在GitHub上已经存在一个仓库了，上图只是示例。

## Gitee上创建对应仓库

在Gitee上创建一个对应的仓库，最好同名：

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/01/23/640-20210123190812594-190812.jpg)git

在创建时，除了填写必要信息之后，最下面一栏选择“导入已有仓库”，然后将GitHub上的仓库地址（Https形式）copy过来，直接粘贴在对应位置。Gitee会检测并给出提示。

点击创建，稍等片刻，发现仓库已经被完美同步过来了：

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/01/23/640-20210123190813268-190813.jpg)git

我的关于shiro的一个仓库地址：[https://gitee.com/secbro/shiro](https://gitee.com/secbro/shiro) 。访问会发现，不仅同步过来了，而且在GitHub上无法显示的图片瞬间显示出来了。

从此刻起，你的开源项目曝光率轻松增加了一倍，是不是很简单而又很有成就感。

## 手动更新同步

经过上面的步骤虽然已经完成了库的同步操作，但你是否发现，当你提交代码到GitHub上时，Gitee上并没有把修改的代码同步过来。

此时可以有两种方案，先说第一种，手动同步。当GitHub上的代码更新了，登录Gitee在项目名称处点击下图中的图标，即可强制同步更新：

![Image](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2021/01/23/640-20210123190813838-190813.jpg)git

这种操作适合非实时同步，可能隔一段时间自己登录账号进行一次同步。

## Git提交同步

第二种方式是Git同步提交多个仓库，这里以Mac操作系统为例，其他操作系统搭建对应找一下相关的命令和操作。

首先，将GitHub的仓库clone到本地，比如执行以下命令：

```text
git clone git@github.com:secbr/shiro.git
```

然后进入本地项目的根目录，在根目录下会有一个.git的隐藏目录。

```text
192:shiro zzs$ ls .git
COMMIT_EDITMSG ORIG_HEAD description info  packed-refs
FETCH_HEAD branches hooks  logs  refs
HEAD  config  index  objects
```

找到.git下面的config文件，通过vi命令进行修改，笔者原本文件内容如下：

```text
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        ignorecase = true
        precomposeunicode = true
[remote "origin"]
        url = git@github.com:secbr/shiro.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
        remote = origin
        merge = refs/heads/main
```

修改之后变为：

```text
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        ignorecase = true
        precomposeunicode = true
[remote "origin"]
        // github的仓库地址
        url = git@github.com:secbr/shiro.git
        // gitee的仓库地址
        url = git@gitee.com:secbro/shiro.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
        remote = origin
        merge = refs/heads/main
```

也就是说，在原来的github仓库地址下面再添加一个url配置，指向gitee的地址。

当然，这里有一个前提条件，Gitee和GitHub的账号的公私钥为同一套。

此时再修改本地代码，进行提交，你会发现GitHub和Gitee上的代码同时被修改了。是不是很cool？

## 小结

很多时候，我们都在用“假勤奋”来麻痹自己，看起来很努力的样子。比如当遇到上述同步代码的问题，如果不深入研究一下，只是通过手动的形式来搬代码，虽然看起来很勤奋，但是比不上一行配置，一行命令。

当然，上述实例只是在GitHub和Gitee两个仓库同步代码，除此之外还可以在GitLab、Bitbucket或是自己搭建的Git服务器上用同样的方式同步代码。

不过，建议根据具体情况而定，最好不要超过两个，不然提交仓库的时候可能会因为网络原因导致耗时比较长。

