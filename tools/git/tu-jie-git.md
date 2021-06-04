# ⭐ 图解Git

[http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg)



此页图解git中的最常用命令。如果你稍微理解git的工作原理，这篇文章能够让你理解的更透彻。 如果你想知道这个站点怎样产生，请前往[GitHub repository](http://github.com/MarkLodato/visual-git-guide)。

## 正文

1. [基本用法](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#basic-usage)
2. [约定](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#conventions)
3. 命令详解
   1. [Diff](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#diff)
   2. [Commit](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#commit)
   3. [Checkout](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#checkout)
   4. [Detached HEAD\(匿名分支提交\)](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#detached)
   5. [Reset](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#reset)
   6. [Merge](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#merge)
   7. [Cherry Pick](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#cherry-pick)
   8. [Rebase](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#rebase)
4. [技术说明](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#technical-notes)

## 基本用法

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/basic-usage.svg-213248.png)

上面的四条命令在工作目录、暂存目录\(也叫做索引\)和仓库之间复制文件。

* `git add *files*` 把当前文件放入暂存区域。
* `git commit` 给暂存区域生成快照并提交。
* `git reset -- *files*` 用来撤销最后一次`git add *files*`，你也可以用`git reset` 撤销所有暂存区域文件。
* `git checkout -- *files*` 把文件从暂存区域复制到工作目录，用来丢弃本地修改。

你可以用 `git reset -p`, `git checkout -p`, or `git add -p`进入交互模式。

也可以跳过暂存区域直接从仓库取出文件或者直接提交代码。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/basic-usage-2.svg-213249.png)

* `git commit -a`相当于运行 `git add` 把所有当前目录下的文件加入暂存区域再运行。`git commit`.
* `git commit *files*` 进行一次包含最后一次提交加上工作目录中文件快照的提交。并且文件被添加到暂存区域。
* `git checkout HEAD -- *files*` 回滚到复制最后一次提交。

## 约定

后文中以下面的形式使用图片。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/conventions.svg-213249.png)

绿色的5位字符表示提交的ID，分别指向父节点。分支用橘色显示，分别指向特定的提交。当前分支由附在其上的_HEAD_标识。 这张图片里显示最后5次提交，_ed489_是最新提交。 _main_分支指向此次提交，另一个_stable_分支指向祖父提交节点。

## 命令详解

### Diff

有许多种方法查看两次提交之间的变动。下面是一些示例。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/diff.svg-213249.png)

### Commit

提交时，git用暂存区域的文件创建一个新的提交，并把此时的节点设为父节点。然后把当前分支指向新的提交节点。下图中，当前分支是_main_。 在运行命令之前，_main_指向_ed489_，提交后，_main_指向新的节点_f0cec_并以_ed489_作为父节点。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/commit-main.svg-213249.png)

即便当前分支是某次提交的祖父节点，git会同样操作。下图中，在_main_分支的祖父节点_stable_分支进行一次提交，生成了_1800b_。 这样，_stable_分支就不再是_main_分支的祖父节点。此时，[合并](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#merge) \(或者 [衍合](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#rebase)\) 是必须的。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/commit-stable.svg-213250.png)

如果想更改一次提交，使用 `git commit --amend`。git会使用与当前提交相同的父节点进行一次新提交，旧的提交会被取消。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/commit-amend.svg-213250.png)

另一个例子是[分离HEAD提交](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#detached),后文讲。

### Checkout

checkout命令用于从历史提交（或者暂存区域）中拷贝文件到工作目录，也可用于切换分支。

当给定某个文件名（或者打开-p选项，或者文件名和-p选项同时打开）时，git会从指定的提交中拷贝文件到暂存区域和工作目录。比如，`git checkout HEAD~ foo.c`会将提交节点_HEAD~_\(即当前提交节点的父节点\)中的`foo.c`复制到工作目录并且加到暂存区域中。（如果命令中没有指定提交节点，则会从暂存区域中拷贝内容。）注意当前分支不会发生变化。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/checkout-files.svg-213250.png)

当不指定文件名，而是给出一个（本地）分支时，那么_HEAD_标识会移动到那个分支（也就是说，我们“切换”到那个分支了），然后暂存区域和工作目录中的内容会和_HEAD_对应的提交节点一致。新提交节点（下图中的a47c3）中的所有文件都会被复制（到暂存区域和工作目录中）；只存在于老的提交节点（ed489）中的文件会被删除；不属于上述两者的文件会被忽略，不受影响。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/checkout-branch.svg-213250.png)

如果既没有指定文件名，也没有指定分支名，而是一个标签、远程分支、SHA-1值或者是像_main~3_类似的东西，就得到一个匿名分支，称作_detached HEAD_（被分离的_HEAD_标识）。这样可以很方便地在历史版本之间互相切换。比如说你想要编译1.6.6.1版本的git，你可以运行`git checkout v1.6.6.1`（这是一个标签，而非分支名），编译，安装，然后切换回另一个分支，比如说`git checkout main`。然而，当提交操作涉及到“分离的HEAD”时，其行为会略有不同，详情见在[下面](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#detached)。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/checkout-detached.svg-213251.png)

### HEAD标识处于分离状态时的提交操作

当_HEAD_处于分离状态（不依附于任一分支）时，提交操作可以正常进行，但是不会更新任何已命名的分支。\(你可以认为这是在更新一个匿名分支。\)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/commit-detached.svg-213251.png)

一旦此后你切换到别的分支，比如说_main_，那么这个提交节点（可能）再也不会被引用到，然后就会被丢弃掉了。注意这个命令之后就不会有东西引用_2eecb_。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/checkout-after-detached.svg-213251.png)

但是，如果你想保存这个状态，可以用命令`git checkout -b *name*`来创建一个新的分支。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/checkout-b-detached.svg-213252.png)

### Reset

reset命令把当前分支指向另一个位置，并且有选择的变动工作目录和索引。也用来在从历史仓库中复制文件到索引，而不动工作目录。

如果不给选项，那么当前分支指向到那个提交。如果用`--hard`选项，那么工作目录也更新，如果用`--soft`选项，那么都不变。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/reset-commit.svg-213252.png)

如果没有给出提交点的版本号，那么默认用_HEAD_。这样，分支指向不变，但是索引会回滚到最后一次提交，如果用`--hard`选项，工作目录也同样。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/reset.svg-213252.png)

如果给了文件名\(或者 `-p`选项\), 那么工作效果和带文件名的[checkout](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#checkout)差不多，除了索引被更新。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/reset-files.svg-213252.png)

### Merge

merge 命令把不同分支合并起来。合并前，索引必须和当前提交相同。如果另一个分支是当前提交的祖父节点，那么合并命令将什么也不做。 另一种情况是如果当前提交是另一个分支的祖父节点，就导致_fast-forward_合并。指向只是简单的移动，并生成一个新的提交。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/merge-ff.svg-213253.png)

否则就是一次真正的合并。默认把当前提交\(_ed489_ 如下所示\)和另一个提交\(_33104_\)以及他们的共同祖父节点\(_b325c_\)进行一次[三方合并](http://en.wikipedia.org/wiki/Three-way_merge)。结果是先保存当前目录和索引，然后和父节点_33104_一起做一次新提交。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/merge.svg-213253.png)

### Cherry Pick

cherry-pick命令"复制"一个提交节点并在当前分支做一次完全一样的新提交。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/cherry-pick.svg-213253.png)

### Rebase

衍合是合并命令的另一种选择。合并把两个父分支合并进行一次提交，提交历史不是线性的。衍合在当前分支上重演另一个分支的历史，提交历史是线性的。 本质上，这是线性化的自动的 [cherry-pick](http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg#cherry-pick)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/rebase.svg-213254.png)

上面的命令都在_topic_分支中进行，而不是_main_分支，在_main_分支上重演，并且把分支指向新的节点。注意旧提交没有被引用，将被回收。

要限制回滚范围，使用`--onto`选项。下面的命令在_main_分支上重演当前分支从_169a6_以来的最近几个提交，即_2c33a_。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/06/04/rebase-onto.svg-213254.png)

同样有`git rebase --interactive`让你更方便的完成一些复杂操作，比如丢弃、重排、修改、合并提交。没有图片体现这些，细节看这里:[git-rebase\(1\)](http://www.kernel.org/pub/software/scm/git/docs/git-rebase.html#_interactive_mode)

## 技术说明

文件内容并没有真正存储在索引\(_.git/index_\)或者提交对象中，而是以blob的形式分别存储在数据库中\(_.git/objects_\)，并用SHA-1值来校验。 索引文件用识别码列出相关的blob文件以及别的数据。对于提交来说，以树\(_tree_\)的形式存储，同样用对于的哈希值识别。树对应着工作目录中的文件夹，树中包含的 树或者blob对象对应着相应的子目录和文件。每次提交都存储下它的上一级树的识别码。

如果用detached HEAD提交，那么最后一次提交会被the reflog for HEAD引用。但是过一段时间就失效，最终被回收，与`git commit --amend`或者`git rebase`很像。

## Walkthrough: Watching the effect of commands

The following walks you through changes to a repository so you can see the effect of the command in real time, similar to how [Visualizing Git Concepts with D3](http://onlywei.github.io/explain-git-with-d3/#) simulates them visually. Hopefully you find this useful.

Start by creating some repository:

```text
$ git init foo
$ cd foo
$ echo 1 > myfile
$ git add myfile
$ git commit -m "version 1"
```

Now, define the following functions to help us show information:

```text
show_status() {
  echo "HEAD:     $(git cat-file -p HEAD:myfile)"
  echo "Stage:    $(git cat-file -p :myfile)"
  echo "Worktree: $(cat myfile)"
}

initial_setup() {
  echo 3 > myfile
  git add myfile
  echo 4 > myfile
  show_status
}
```

Initially, everything is at version 1.

```text
$ show_status
HEAD:     1
Stage:    1
Worktree: 1
```

We can watch the state change as we add and commit.

```text
$ echo 2 > myfile
$ show_status
HEAD:     1
Stage:    1
Worktree: 2
$ git add myfile
$ show_status
HEAD:     1
Stage:    2
Worktree: 2
$ git commit -m "version 2"
[main 4156116] version 2
 1 file changed, 1 insertion(+), 1 deletion(-)
$ show_status
HEAD:     2
Stage:    2
Worktree: 2
```

Now, let's create an initial state where the three are all different.

```text
$ initial_setup
HEAD:     2
Stage:    3
Worktree: 4
```

Let's watch what each command does. You will see that they match the diagrams above.

`git reset -- myfile` copies from HEAD to stage:

```text
$ initial_setup
HEAD:     2
Stage:    3
Worktree: 4
$ git reset -- myfile
Unstaged changes after reset:
M   myfile
$ show_status
HEAD:     2
Stage:    2
Worktree: 4
```

`git checkout -- myfile` copies from stage to worktree:

```text
$ initial_setup
HEAD:     2
Stage:    3
Worktree: 4
$ git checkout -- myfile
$ show_status
HEAD:     2
Stage:    3
Worktree: 3
```

`git checkout HEAD -- myfile` copies from HEAD to both stage and worktree:

```text
$ initial_setup
HEAD:     2
Stage:    3
Worktree: 4
$ git checkout HEAD -- myfile
$ show_status
HEAD:     2
Stage:    2
Worktree: 2
```

`git commit myfile` copies from worktree to both stage and HEAD:

```text
$ initial_setup
HEAD:     2
Stage:    3
Worktree: 4
$ git commit myfile -m "version 4"
[main 679ff51] version 4
 1 file changed, 1 insertion(+), 1 deletion(-)
$ show_status
HEAD:     4
Stage:    4
Worktree: 4
```





