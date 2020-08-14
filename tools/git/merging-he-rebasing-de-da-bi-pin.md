# Merging 和 Rebasing 的大比拼

[https://mp.weixin.qq.com/s/rbWZbTsRMhWtoTrLtDie0A](https://mp.weixin.qq.com/s/rbWZbTsRMhWtoTrLtDie0A)

当 merging 和rebasing 在 git 中相似时，他们提供不同的功能。为了让你的历史尽可能的干净和完整，你应该知道以下几点。

`git rebase` 命令已 神奇的 Git voodoo 而闻名，初学者应该远离它，但它实际上可以让开发团队在使用时更加轻松。在本章中，我们将 把 `git rebase` 和与之有关联的 `git merge` 命令相比较 ，并在典型的 Git 工作流 中重新定位，识别其所有潜在的机会。

## 概述

首先要明白关于 `git rebase` 的事情是它像 `git merge` 一样解决相同的问题。git rebase 和 git merge 一样都是被设计用于从一个分支获取并合并到当前分支，但是他们采取不同的工作方式。

考虑一下，当你开始在 一个专用的分支上开发新特性，与此同时另一个团队成员用新的提交来更新了 `master` 分支时，会发生什么呢？这会导致分叉的历史记录，对于这个问题，使用 Git 作为协同工具的任何人来说都应该很熟悉。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729180132610-180132.jpg)

现在，假设你在工作时在 `master` 上的新提交与新特性相关。为了将新提交合并到你的 `feature` 分支上，你有两种选择：merging 或者 rebasing。

### Merge 选项

最简单的选项是使用以下命令将 `master` 分支合并到 `feature` 分支：

```text
git checkout feature
git merge master
```

或者，你可以简化成一句：

```text
git merge master feature
```

这将在 `feature` 分支上创建一个新 “ 合并提交 ” ，并把两个分支的历史联系在一起。分支结构显示如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729180132816-180132.jpg)

Merging 之所以好是因为它是一个不可逆的操作。在任何情况下，现有分支不能被更改。这避免了所有 rebasing 的潜在陷阱（详见下文）。

另一方面，这也意味着每次需要合并上游更改时， `feature` 分支都将有一个额外的 merge 提交产生。如果 `master` 非常活跃，这可能破坏你全部的 feature 分支的历史。使用高级的 `git log` 选项来减缓这个问题是有可能的，也让其他开发人员很难理解这个项目的历史记录。

### Rebase  选项

作为 merging 的一个替代品，你可以使用以下命令将 `feature` 分支合并到 `master` 分支：

```text
git checkout feature
git rebase master
```

这将整个 `feature` 分支从 `master` 分支的顶端开始，有效地将所有新的提交合并到主分支中。但是，并不是使用合并提交，而是通过为每个在原始分支上的提交创建全新的提交来重写项目历史。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729180133062-180133.jpg)

rebasing 最主要的益处是你将获得一个十分干净整洁的项目历史。首先，它通过 `git merge` 排除多余的 merge 提交需求；其次，正如你在上图所看到的那样，rebasing 也会产生完美线性的项目历史记录—你可以顺着 `feature` 一直到项目的起始位置而没有任何分支。可以方便的使用 `git log` ，`git bisect` 和 `gitk` 追踪提交记录。

但是，对于新的提交历史有两点需要权衡：安全性和可追溯性。如果你不遵循 Rebasing 的黄金法则，为你的协作工作流重写项目历史可能会成为潜在的灾难。另外，不重要的是，rebasing 会丢失合并提交所提供的上下文—你不能看到何时合并到 feature 分支中的上游变化。

### 交互式的 Rebasing

当他们移动到新的分支上，交互式合并给你机会来修改提交。自从它提供完全控制整个分支的提交历史之后，它比自动合并更强大。具有代表性的，在合并一个 feature 分支到 `master` 时，它是被用来清除错误的历史。

要开始一个交互式的重基会话，请将 `i` 选项传递给 `git rebase` 命令:

```text
git checkout feature
git rebase -i master
```

这将打开一个文本编辑器列出所有要被移动的提交：

```text
pick 33d5b7a Message for commit #1
pick 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```

此列表准确定义了执行 rebase 后分支的外观。通过改变 `pick` 命令或调整条目顺序来改变分支的提交历史，你可以让分支看起来像任何你想要的样子。举例说，如果第二次提交是为了修复第一次提交中的一个小问题，你可以使用 `fixup` 命令把他们简化成一个简单的命令：

```text
pick 33d5b7a Message for commit #1
fixup 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```

当你保存并关闭文件时，Git 将根据你的指令来执行 rebase ，从而产生如下所示的项目历史记录：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729180133338-180133.jpg)

像这样排除不重要的提交使你的特性历史相当易懂。这一点是 `git merge` 无法比拟的。

## Rebasing 的黄金规则

一旦你明白什么是 rebasing ，最重要的事情是学习什么时候不用它。`git rebase` 的黄金法则是永远不要在公有分支上使用它。

举例说，想象一下如果你将 `master` 分支合并到 `feature` 分支上会发生什么：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729180133572-180133.jpg)

rebase 操作将 `master` 中所有提交移动到 `feature` 的头部，但问题是这一切都发生在你的仓库中。其他开发者依然在原来的 `master` 分支上继续工作。自从 rebasing 产生了全新的提交，Git 将会认为你的 `master` 分支的历史记录与其他人的历史记录不同。

使两个 `master` 分支 同步的唯一方法是将他们合并到一起，导致出现一个额外的合并操作和两组都包含相同改变（最原始的那个，和那些来自你重新建立的分支）的提交。不用说，这是一个非常混乱的场景。

因此，在你运行 `git rebase` 之前，一定要问自己，“还有其他人在看这个分支吗？”，如果回答是肯定的，那么把你的手从键盘上拿开并开始考虑让你的改变没有破坏性（例如， `git revert` 命令）。否则，你可以随心所欲地重写历史。

### Force-Pushing

如果你尝试将合并的 `master` 分支推送到远程库中，Git 将防止你这样做，因为它与远程 `master` 分支有冲突。但是，你可以通过传递 `--force` 标志来强制推送，就像这样：

```text
# Be very careful with this command!
git push --force
```

该操作会将远程仓库的 `master` 分支替换为 rebase 过的 `master` 分支，这会给团队的其他成员带来困扰。因此，当你确切的知道你要做什么的时候，才要非常小心的使用这些命令。

推送一个私有新特性分支到远程仓库（例如，用于备份）。这就好像是说，“哎呦，我不想推送 feature 分支的原始版本，拿当前的版本替换吧。”再强调一次，没有人在 feature 分支的原始版本中工作是很重要的。

## 工作流演练

Rebasing 能够根据团队的需要或多或少的被合并到你现存的 Git 工作流 中。在这个选项中，我们将检查 rebasing 提供在不同阶段的 feature 分支开发的好处。

在任何工作流中，首先第一步是利用 `git rebase` 为每一个 feature 创建一个专用的分支。这给你必要分支结构来安全使用 rebasing ：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729180133757-180133.jpg)

### 本地清除

最好的方法之一是合并 rebasing 到你的 工作流 以此来清理本地正在进行的 feature 分支。通过定期的执行一个交互式的 rebase ，你可以确保每一个在你的 feature 分支中的提交是集中且有意义的。这将让你编写你自己的代码而不需要在独立提交中担心破坏它—你可以在事后修复它。

当调用 `git rebase` ，对于新的分支你有两个选项：feature 父类分支（举例说，`master` 分支），或者在你的 feature 分支中较早的提交。我们查看了在 _交互式的 Rebasing_ 章节中首个选项的示例 。当你仅仅需要修复最新提交时，后者的选择最好。举例说，交互式 rebase 的最后3次提交显示如下：

```text
git checkout feature
git rebase -i HEAD~3
```

通过指定 `HEAD~3` 作为新的基础，事实上你并没有移动分支—你只是交互式的重写了接下来的3次提交。请注意，这不会将上游更改合并到 `feature` 分支。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729180134032-180134.jpg)

如果你想使用这个方法重写整个 feature， `git merge-base` 命令对于找到 `feature` 分支的原始起始点非常有用。以下返回原始起始点的提交 ID ，然后传递给 `git rebase` ：

```text
git merge-base feature master
```

交互式 rebasing 的作用在于当他仅仅影响本地分支时，它是一个 引进 `git rebase` 到工作流中的好方式。其他开发人员唯一能看到的是你最后提交的成果，这应该是一个简单且易于理解的 feature 分支历史记录。

但是在刚开始，这仅仅只为私有 feature 分支工作。如果你借助相同 feature 分支与其他开发者协作，分支是共有的，你也不被允许重写它的历史记录。

没有 `git merge` 之外的其他选择时可以使用交互式 rebase 来清除本地提交。

### 合并上游更改到 Feature 中

在开篇章节中，我们知道了 feature 分支如何使用 `git merge` 或 `git rebase` 合并 `master` 分支的上游提交。当 rebasing 通过移动你的 feature 分支到 `master` 分支的头部来创建一个线性历史时，Merging 是一个用于保护你仓库的整个历史记录的安全选项。

`git rebase` 的作用与本地清除相似（能够同时被执行），但是在此过程中，它合并了 `master` 的上游提交。

牢记，远程分支取代 `master` 分支是完全合法的。这发生在其他开发者在同一个 feature 分支上协作时和你需要合并他们的更改到你的仓库中时。

举例说明，如果你和一个名为 John 的开发人员添加了对 `feature` 分支的提交，从 John 的仓库中获取远程 `feature` 分支后，你的仓库看起来像如下所示：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729180134216-180134.jpg)

你可以用与 `master` 分支集成上游更改相同的方法来解决这个分叉：或者你本地的 `feature` 分支与 `john/feature` 分支合并，或者 rebase 你本地 `feature` 分支到 `john/feature` 分支的头部。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729180134445-180134.jpg)

请注意，任何事情在未更改之前，rebase 不能违反 _Rebasing 的黄金法则_ ，因为 `feature` 仅仅移动了本地提交。这就好像是在说，“将我的更改添加到 John 已经完成了的操作中。” 在大多数情况下，这比通过合并提交与远程分支同步更为直观。

默认情况下， `git pull` 命令执行合并，但是你可以强制通过使用 rebase 的 `--rebase` 选项整合远程分支。

### 使用 Pull 请求检验 feature 分支

如果你使用 Pull 请求作为代码的审计过程，创建的 pull 请求之后，你需要避免使用 `git rebase` 。一旦你发出 pull 请求，其他开发人员就能看到你的提交，这就意味着它是一个公有分支。重写它的历史记录将使 Git 和你的队友无法追踪到任何添加到 feature 分支上的后续提交。

任何来自其他开发者的更改需要使用 `git merge` 取代 `git rebase` 来被合并。

为此，在提交你的 pull 请求之前，使用交互式 rebase 清理你的代码，通常是一个好主意。

### 整合认可的 feature

在 feature 分支被你的团队认可之后，在使用 `git merge` 整合 feature 分支到主代码库之前，你有一个 rebasing feature 分支到 `master` 分支的选项。

合并上游更改到 feature 分支是一个类似的情况，但是，自从你不被允许在 `master` 中重写提交，你最后不得不使用 `git merge` 来整合 feature 分支。然而，通过在合并之前执行 rebase 确保 merge 将快速进行，形成完美的线性历史。这也给了你在 pull 请求期间将任何后续提交塞入到 feature 分支中的机会。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729180134746-180134.jpg)

如果你对 `git rebase` 感到不太舒服，你可以在临时分支中一直执行 rebase。那样，如果你一不小心搞砸了你的 feature 分支历史记录，你可以多次检查原始分支。例如：

```text
git checkout feature
git checkout -b temporary-branch
git rebase -i master
# [Clean up the history]
git checkout master
git merge temporary-branch
```

## 总结

在你开始 rebasing 你的分支之前，这是所有你真正需要知道：如果您想要一个没有不必要的干净的合并提交的线性历史记录，你应该争取 `git rebase` 代替 `git merge` 整合来自另一个分支的改变。

另一方面，如果你想保存你项目的完整历史并且避免重写公有提交的风险，你可以坚持使用 `git merge` 。任何一个选项都是完全有效的，至少现在你是有选择性的利用 `git rebase` 的好处。

> 原文：[https://dzone.com/articles/merging-vs-rebasing](https://dzone.com/articles/merging-vs-rebasing)
>
> 原文作者：Tim Pettersen
>
> 译者：ねこ
>
> 参考文章：[https://www.jianshu.com/p/f23f72251abc](https://www.jianshu.com/p/f23f72251abc)
>
> [https://www.cnblogs.com/FraserYu/p/11192840.html](https://www.cnblogs.com/FraserYu/p/11192840.html)
>
> [https://www.html.cn/archives/10077](https://www.html.cn/archives/10077)
>
> [https://www.jianshu.com/p/318b4f0f57ff](https://www.jianshu.com/p/318b4f0f57ff)

