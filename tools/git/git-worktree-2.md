# git worktree 2



[https://mp.weixin.qq.com/s/6nCCy2SaYz44WoM\_3Xw6LQ](https://mp.weixin.qq.com/s/6nCCy2SaYz44WoM\_3Xw6LQ)





> [https://www.blopig.com/blog/2017/06/using-bare-git-repos/\
> https://lists.mcs.anl.gov/pipermail/petsc-dev/2021-May/027448.html\
> https://www.saintsjd.com/2011/01/what-is-a-bare-git-repository/\
> https://infrequently.org/2021/07/worktrees-step-by-step/\
> https://morgan.cugerone.com/blog/how-to-use-git-worktree-and-in-a-clean-way/](https://www.blopig.com/blog/2017/06/using-bare-git-repos/https://lists.mcs.anl.gov/pipermail/petsc-dev/2021-May/027448.htmlhttps://www.saintsjd.com/2011/01/what-is-a-bare-git-repository/https://infrequently.org/2021/07/worktrees-step-by-step/https://morgan.cugerone.com/blog/how-to-use-git-worktree-and-in-a-clean-way/)



要想生成一个 `bare repo` 也很简单, 只需在上面两个命令的基础上加上 `--bare` 参数即可

```
git init --baregit clone --bare https://github.com/FraserYu/amend-crash-demo.git
```

来执行这两个 clone 命令，并查看文件内容你就会看出差别了

![](https://mmbiz.qpic.cn/mmbiz\_png/WGOSCfqtNHpo5bHLiauSB0tdice1IoJtkNrictSknl1yoqucBbZjdCDd4Rzbh8x7MUMXby4Q5Mm5krmJru7Da4h5Q/640?wx\_fmt=png\&tp=webp\&wxfrom=5\&wx\_lazy=1\&wx\_co=1)

1. `bare repo` 仅仅包含 Git 相关信息，并不包含我们的实际项目文件（`.java`/`.js`/`.py`）,  而 `non-bare repo` 却包含我们的全部信息
2. bare repo 名称默认是带有 `.git` 后缀的，这也恰恰证明了第一点
3. bare repo 中是不存在 `.git` 文件夹的，这也就导致它不能像 `non-bare repo` 那样 `add`/`commit`/`pull`/`push`

看到这，你可能感觉 bare repo 就是一个 Git 空壳文件夹，一无是处。其实不然，正因为 bare repo 的这些特性（不能对它进行更改），也就避免 repo 里面的内容被弄的一团糟，所以可以被用来做私有的中心化 repo，一张图解释，其实就是这样的：

![](https://mmbiz.qpic.cn/mmbiz\_png/WGOSCfqtNHpo5bHLiauSB0tdice1IoJtkNEz5vGYAH3CQfGkF8j7Zpwm94tVyvrsib9ibOMDSL0v5NFVDoBvrv1jOg/640?wx\_fmt=png\&tp=webp\&wxfrom=5\&wx\_lazy=1\&wx\_co=1)

如果你有兴趣，可以按照下面的命令在本地实验一下整个过程：

```
user@server:$~  git init --bare name_to_repo.gituser@machine1:$~ git clone user@server:/path/to/repo/name_to_repo.git .user@machine1:$~ cd name_to_repouser@machine1:$~ touch READMEuser@machine1:$~ echo "Hello world" >> READMEuser@machine1:$~ git add READMEuser@machine1:$~ git commit -m "Adding a README"user@machine1:$~ git push origin masteruser@server:$~ ls /path/to/repo/name_to_repo.git/branches/ config description HEAD hooks/ info/ objects/ refs/user@machine2:$~ git clone user@server:/path/to/repo/name_to_repo.git .user@machine2:$~ ls name_to_repo.git/READMEuser@machine2:$~ cat READMEHello world
```

> 实验结果就是：无论在 machine1 下怎么 push 文件，bare repo 中都不会存在你 push 的文件，只有 Git 相关信息。但是 machine2 clone 最新的 repo 时，却能看到machine1 push 的文件，这正是 Git 的魔法所在

到这里，bare repo 就解释完了。接下来，接上一篇  Git Worktree 大法真香 内容，借助 bare repo 的特性，来优化同时在多个分支工作的做法吧

### Git Worktree 高级用法

首先，在你选定的目录下为你的项目（比如这里叫 `amend-crash-demo`）单独创建一个文件夹, 并 `cd` 进去

```
mkdir amend-crash-democd amend-crash-demo
```

接下来以 bare 的形式 clone 项目代码, 并将内容 clone 到 `.bare` 文件夹内：

```
git clone --bare git@github.com:FraserYu/amend-crash-demo.git .bare
```

我们还要在当前目录下创建一个 `.git` 文件，文件内容是以 `gitdir` 的形式指向我们的 `.bare` 文件夹 （如果不理解 gitdir 的含义，请回看  Git Worktree 大法真香 ）

```
echo "gitdir: ./.bare" > .git
```

然后我们要编辑 .bare/config 文件，并修改 `[remote "origin"]`内容，和下面内容保持一致（也就是添加第 6 行内容），这确保我们创建 worktree 切换分支，可以显示正确的分支名称

```
vim .bare/config# ----------------------------------------------- [remote "origin"]       url = git@github.com:FraserYu/amend-crash-demo.git       fetch = +refs/heads/*:refs/remotes/origin/*
```

接下来我们就可以创建 worktree 了，首先我们要为 main 分支创建 worktree，因为 main 分支 HEAD 的指向的 commit-ish 就是你创建其他 worktree 的 commit-ish

```
git worktree add main# --------------------------------Preparing worktree (checking out 'main')HEAD is now at 82b8711 add main file
```

通常我们不会直接在 main 分支上直接工作，而是创建其它类型的分支，继续创建名为 `feature/JIRA234-feature3` 的 worktree

```
git worktree add -b "feature/JIRA234-feature3" feature3# ------------------------------------------------Preparing worktree (new branch 'feature/JIRA234-feature3')HEAD is now at 82b8711 add main file
```

查看当前文件夹的内容，你会发现只有 `main` 和 `feature3` 两个文件夹（因为 `.bare` 和 `.git` 是隐藏文件夹/文件），这样是不是相当清爽呢？

```
ls -l# -------------------------------------------total 0drwxr-xr-x  10  rgyb  staff  320 Nov 23 21:44 feature3drwxr-xr-x  10  rgyb  staff  320 Nov 23 21:36 mainls -al# -------------------------------------------total 8drwxr-xr-x   6  rgyb  staff  192 Nov 23 21:44  .drwxr-xr-x   3  rgyb  staff   96 Nov 23 21:14   ..drwxr-xr-x  12  rgyb  staff  384 Nov 23 21:36  .bare-rw-r--r--   1  rgyb  staff   16 Nov 23 21:29   .gitdrwxr-xr-x  10  rgyb  staff  320 Nov 23 21:44  feature3drwxr-xr-x  10  rgyb  staff  320 Nov 23 21:36  main
```

接下来就可以尽情的在我们的各种分支上，彼此互不影响的进行  `add`/`commit`/`pull`/`push` 操作了

```
echo "feature3 development" > feature3.yamlgit add feature3.yamlgit commit -m "feat: [JIRA234-feature3] feature3 development"# ------------------------------------------------------------------[feature/JIRA234-feature3 aeaac94] feat: [JIRA234-feature3] feature3 development 1 file changed, 1 insertion(+) create mode 100644 feature3.yamlgit push --set-upstream origin feature/JIRA234-feature3
```

通过上一篇文章 worktree 的四个命令，多分支协同开发不再是问题：

```
git worktree addgit worktree list# ------------------------------------------------------------------------------------------------/Users/rgyb/Documents/projects/amend-crash-demo/.bare        (bare)/Users/rgyb/Documents/projects/amend-crash-demo/feature3   aeaac94 [feature/JIRA234-feature3]/Users/rgyb/Documents/projects/amend-crash-demo/main        82b8711 [main]git worktree removegit worktree prune
```

### 总结

通过借助 bare repo 的特性，我们可以非常整洁的将所有 worktree 只管理在当前项目目录下，多分支协同开发，就像这样：

```
.└── amend-crash-demo    ├── feature3    │   ├── README.md    │   ├── config.yaml    │   ├── feat1.txt    │   ├── feature3.yaml    │   ├── file1.yaml    │   ├── hotfix.yaml    │   ├── main.properties    │   └── main.yaml    └── main        ├── README.md        ├── config.yaml        ├── feat1.txt        ├── file1.yaml        ├── hotfix.yaml        ├── main.properties        └── main.yaml3 directories, 15 files
```

如果你有磁盘管理强迫症，这绝对是个好办法。

> 如果你想更好的理解整个过程，你需要在操作本文命令的同时，查看 Git 相关的文件信息

有什么问题，留言区交流

### 参考

1. https://www.blopig.com/blog/2017/06/using-bare-git-repos/
2. https://lists.mcs.anl.gov/pipermail/petsc-dev/2021-May/027448.html
3. https://www.saintsjd.com/2011/01/what-is-a-bare-git-repository/
4. https://infrequently.org/2021/07/worktrees-step-by-step/
5. https://morgan.cugerone.com/blog/how-to-use-git-worktree-and-in-a-clean-way/





