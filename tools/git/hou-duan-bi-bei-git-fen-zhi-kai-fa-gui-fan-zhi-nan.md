# 后端必备 Git 分支开发：规范指南

[https://mp.weixin.qq.com/s?\_\_biz=MzUxOTc4NjEyMw==&mid=2247492196&idx=2&sn=eedb873b6f4049cf2df5e486f978d6e1&chksm=f9f6f980ce8170969c96f32b71771781315680ee1e1e2eb5467272150ad371b70b72473c3f25&mpshare=1&scene=1&srcid=0826FCSK3yBAcvu2wEszG8Qc&sharer\_sharetime=1598419999857&sharer\_shareid=393f249533d421d13c2402bd43e74356\#rd](https://mp.weixin.qq.com/s?__biz=MzUxOTc4NjEyMw==&mid=2247492196&idx=2&sn=eedb873b6f4049cf2df5e486f978d6e1&chksm=f9f6f980ce8170969c96f32b71771781315680ee1e1e2eb5467272150ad371b70b72473c3f25&mpshare=1&scene=1&srcid=0826FCSK3yBAcvu2wEszG8Qc&sharer_sharetime=1598419999857&sharer_shareid=393f249533d421d13c2402bd43e74356#rd)



* 分支管理
* * 分支命名
    * 常见任务
* 日志规范
* Commit messages的基本语法

Git 是目前最流行的源代码管理工具。为规范开发，保持代码提交记录以及 git 分支结构清晰，方便后续维护，现规范 git 的相关操作。

## 分支管理

### 分支命名

#### master 分支

master 为主分支，也是用于部署生产环境的分支，确保master分支稳定性

master 分支一般由develop以及hotfix分支合并，任何时间都不能直接修改代码

#### develop 分支

develop 为开发分支，始终保持最新完成以及bug修复后的代码

一般开发的新功能时，feature分支都是基于develop分支下创建的

#### feature 分支

开发新功能时，以develop为基础创建feature分支

分支命名: feature/ 开头的为特性分支， 命名规则: feature/user\_module、 feature/cart\_module

#### release分支

release 为预上线分支，发布提测阶段，会release分支代码为基准提测

> “
>
> 当有一组feature开发完成，首先会合并到develop分支，进入提测时，会创建release分支。如果测试过程中若存在bug需要修复，则直接由开发者在release分支修复并提交。当测试完成之后，合并release分支到master和develop分支，此时master为最新代码，用作上线。

#### hotfix 分支

分支命名: hotfix/ 开头的为修复分支，它的命名规则与 feature 分支类似

线上出现紧急问题时，需要及时修复，以master分支为基线，创建hotfix分支，修复完成后，需要合并到master分支和develop分支

### 常见任务

#### 增加新功能

```text
(dev)$: git checkout -b feature/xxx            # 从dev建立特性分支
(feature/xxx)$: blabla                         # 开发
(feature/xxx)$: git add xxx
(feature/xxx)$: git commit -m 'commit comment'
(dev)$: git merge feature/xxx --no-ff          # 把特性分支合并到dev
```

#### 修复紧急bug

```text
(master)$: git checkout -b hotfix/xxx         # 从master建立hotfix分支
(hotfix/xxx)$: blabla                         # 开发
(hotfix/xxx)$: git add xxx
(hotfix/xxx)$: git commit -m 'commit comment'
(master)$: git merge hotfix/xxx --no-ff       # 把hotfix分支合并到master，并上线到生产环境
(dev)$: git merge hotfix/xxx --no-ff          # 把hotfix分支合并到dev，同步代码
```

#### 测试环境代码

```text
(release)$: git merge dev --no-ff             # 把dev分支合并到release，然后在测试环境拉取并测试
```

#### 生产环境上线

```text
(master)$: git merge release --no-ff          # 把release测试好的代码合并到master，运维人员操作
(master)$: git tag -a v0.1 -m '部署包版本名'  #给版本命名，打Tag
```

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/26/640-20200826142620223-142620.jpg)

> “
>
> 推荐一个艿艿写的 6000+ Star 的 SpringBoot + SpringCloud + Dubbo 教程的仓库：[https://github.com/YunaiV/SpringBoot-Labs](https://github.com/YunaiV/SpringBoot-Labs)

## 日志规范

在一个团队协作的项目中，开发人员需要经常提交一些代码去修复bug或者实现新的feature。而项目中的文件和实现什么功能、解决什么问题都会渐渐淡忘，最后需要浪费时间去阅读代码。但是好的日志规范commit messages编写有帮助到我们，它也反映了一个开发人员是否是良好的协作者。

编写良好的Commit messages可以达到3个重要的目的：

加快review的流程

帮助我们编写良好的版本发布日志

让之后的维护者了解代码里出现特定变化和feature被添加的原因

目前，社区有多种 Commit message 的写法规范。来自Angular 规范是目前使用最广的写法，比较合理和系统化。如下图：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/26/640-20200826142620275-142620.jpg)

> “
>
> 推荐一个艿艿写的 3000+ Star 的 SpringCloud Alibaba 电商开源项目的仓库：[https://github.com/YunaiV/onemall](https://github.com/YunaiV/onemall)

## Commit messages的基本语法

当前业界应用的比较广泛的是 Angular Git Commit Guidelines

> “
>
> [https://github.com/angular/angular.js/blob/master/DEVELOPERS.md\#-git-commit-guidelines](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines)

具体格式为:

```text
<type>: <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

type: 本次 commit 的类型，诸如 bugfix docs style 等

scope: 本次 commit 波及的范围

subject: 简明扼要的阐述下本次 commit 的主旨，在原文中特意强调了几点 1. 使用祈使句，是不是很熟悉又陌生的一个词，来传送门在此 祈使句 2. 首字母不要大写 3. 结尾无需添加标点

body: 同样使用祈使句，在主体内容中我们需要把本次 commit 详细的描述一下，比如此次变更的动机，如需换行，则使用 \|

footer: 描述下与之关联的 issue 或 break change，详见案例

### Type的类别说明：

* feat: 添加新特性
* fix: 修复bug
* docs: 仅仅修改了文档
* style: 仅仅修改了空格、格式缩进、都好等等，不改变代码逻辑
* refactor: 代码重构，没有加新功能或者修复bug
* perf: 增加代码进行性能测试
* test: 增加测试用例
* chore: 改变构建流程、或者增加依赖库、工具等

### Commit messages格式要求

```text
# 标题行：50个字符以内，描述主要变更内容
#
# 主体内容：更详细的说明文本，建议72个字符以内。 需要描述的信息包括:
#
# * 为什么这个变更是必须的? 它可能是用来修复一个bug，增加一个feature，提升性能、可靠性、稳定性等等
# * 他如何解决这个问题? 具体描述解决问题的步骤
# * 是否存在副作用、风险?
#
# 如果需要的化可以添加一个链接到issue地址或者其它文档
```

