# Git科普文，Git基本原理&各种骚操作

[_**https://mp.weixin.qq.com/s/csEgAjJwH75\_IvAnFBIuvw**_](https://mp.weixin.qq.com/s/csEgAjJwH75\_IvAnFBIuvw)

**Git简单介绍\*\***

`Git`是一个分布式版本控制软件，最初由`Linus Torvalds`创作，于2005年以`GPL`发布。最初目的是为更好地管理`Linux`内核开发而设计。

##

## _**\\**_**Git工作流程以及各个区域\*\***

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101809665-101809.png)

* Workspace：工作区
* Staging/Index：暂存区
* Local Repository：本地仓库（可修改）
* /refs/remotes：远程仓库的引用（不可修改）
* Remote：远程仓库

##

## _**\\**_**Git文件状态变化\*\***

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101811011-101811.png)

## _**\\**_**Git各种命令\*\***

### Git简单命令

```
# 在当前目录新建一个git仓库
git init

# 打开git仓库图形界面
gitk

# 显示所有变更信息
git status

# 删除所有Untracked files
git clean -fd

# 下载远程仓库的所有更新
git fetch remote

# 下载远程仓库的所有更新，并且Merge
git pull romote branch-name


# 查看上次commit id
git rev-parse HEAD 

# 将指定分支合并到当前分支
git merge branch-name

# 将最近的一次commit打包到patch文件中
git format-patch HEAD^ 

# 将patch文件 添加到本地仓库
git am  patch-file

# 查看指定文件修改历史
git blame file-name
```

### Git常用命令

#### git clone

```
# 将远程git仓库克隆到本地
git clone url

# 将远程git仓库克隆到本地
git clone -b branch url
```

#### git stash

```
# 将修改过，未add到Staging区的文件，暂时存储起来
git stash

# 恢复之前stash存储的内容
git stash apply

# 保存stash 并写message
git stash save "stash test"

# 查看stash了哪些存储
git stash list

# 将stash@{1}存储的内容还原到工作区
git stash apply stash@{1}

# 删除stash@{1}存储的内容
git stash drop stash@{1}

# 删除所有缓存的stash
git stash clear
```

#### git config

```
# 配置git图形界面编码为utf-8
git config --global gui.encoding=utf-8 

# 设置全局提交代码的用户名 
git config --global user.name name  
# 设置全局提交代码时的邮箱
git config --global user.email email
# 设置当前项目提交代码的用户名 
git config user.name name
```

#### git remote

```
# 显示所有远程仓库
git remote -v  

#  增加一个新的远程仓库
git remote add name url 

#  删除指定远程仓库
git remote remove name

# 获取指定远程仓库的详细信息
git remote show origin
```

#### git add

```
# 添加所有的修改到Staging区
git add .
git add --all  

# 添加指定文件到Staging区
git add file   

# 添加多个修改的文件到Staging区
git add file1 file2   

# 添加修改的目录到Staging区
git add dir

# 添加所有src目录下main开头的所有文件到Staging区    
git add src/main*
```

#### git commit

```
# 提交Staging区的代码到本地仓库区
git commit -m "message"  

# 提交Staging中在指定文件到本地仓库区
git commit file1 file2 -m "message"  

# 使用新的一次commit，来覆盖上一次commit
git commit --amend -m "message" 

# 修改上次提交的用户名和邮箱
git commit --amend --author="name <email>" --no-edit
```

#### git branch

```
# 列出本地所有分支
git branch   

# 列出本地所有分支 并显示最后一次提交的哈希值
git branch -v

# 在-v 的基础上 并且显示上游分支的名字
git branch -vv

# 列出上游所有分支
git branch -r  

# 新建一个分支，但依然停留在当前分支
git branch branch-name  

# 删除分支
git branch -d branch-name   

# 设置分支上游
git branch --set-upstream-to origin/master

# 本地分支重命名
git branch -m old-branch new-branch
```

#### git checkout

```
# 创建本地分支并关联远程分支
git checkout -b local-branch origin/remote-branch

# 新建一个分支，且切换到新分支
git checkout -b branch-name

# 切换到另一个分支
git checkout branch-name  

# 撤销工作区文件的修改，跟上次Commit一样
git checkout commit-file
```

#### git tag

```
# 创建带有说明的标签
git tag -a v1.4 -m 'my version 1.4'

#  打标签
git tag tag-name

# 查看所有标签
git tag 

# 给指定commit打标签
git tag tag-name commit-id

# 删除标签
git tag -d tag-name
```

#### git push

```
# 删除远程分支
git push origin :master   

#  删除远程标签
git push origin --delete tag tag-name

# 上传本地仓库到远程分支
git push remote branch-name

# 强行推送当前分支到远程分支
git push remote branch-name --force

# 推送所有分支到远程仓库
git push remote --all  

# 推送所有标签
git push --tags

# 推送指定标签
git push origin tag-name

#  删除远程标签（需要先删除本地标签）
git push origin :refs/tags/tag-name  

# 将本地dev分支push到远程master分支
git push origin dev:master
```

#### git reset

```
# 将未commit的文件移出Staging区
git reset HEAD

# 重置Staging区与上次commit的一样
git reset --hard  

# 重置Commit代码和远程分支代码一样
git reset --hard origin/master

# 回退到上个commit
git reset --hard HEAD^

# 回退到前3次提交之前，以此类推，回退到n次提交之前
git reset --hard HEAD~3

回退到指定commit
git reset --hard commit-id
```

#### git diff

```
# 查看文件在工作区和暂存区区别
git diff file-name

# 查看暂存区和本地仓库区别
git diff --cached  file-name

# 查看文件和另一个分支的区别
git diff branch-name file-name

# 查看两次提交的区别
git diff commit-id commit-id
```

#### git show

```
# 查看指定标签的提交信息
git show tag-name

# 查看具体的某次改动
git show commit-id
```

#### git log

```
# 指定文件夹 log
git log --pretty=format:"%h %cn %s %cd" --author="iisheng\|胜哥"  --date=short src
# 查看指定用户指定format 提交
  git log --pretty=format:"%h %cn %s %cd" --author=iisheng --date=short 

# 查看该文件的改动历史
git log --pretty=oneline file

# 图形化查看历史提交
git log --graph --pretty=oneline --abbrev-commit

# 统计仓库提交排名前5
git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5

# 查看指定用户添加代码行数，和删除代码行数
git log --author="iisheng" --pretty=tformat: --numstat | awk '{ add += $1 ; subs += $2 } END { printf "added lines: %s removed lines : %s \n",add,subs }'
```

#### git rebase

```
# 将指定分支合并到当前分支
git rebase branch-name

# 执行commit id 将rebase 停留在指定commit 处
git rebase -i commit-id

# 执行commit id 将rebase 停留在 项目首次commit处
git rebase -i --root
```

#### git restore

```
# 恢复第一次add 的文件，同 git rm --cached
git restore --staged file

# 移除staging区的文件，同 git checkout
git restore file
```

#### git revert

```
# 撤销前一次commit
git revert HEAD

# 撤销前前一次commit
git revert HEAD^

# 撤销指定某次commit
git revert commit-id
```

##

## _**\\**_**Git骚操作\*\***

#### Git命令不能自动补全？（Mac版）

❝我见过有的人使用`Git`别名，反正因为有自动补全的存在，我从来没用过`Git`别名。不过我的确将我的`rm -rf`命令替换成了别的脚本了...

安装`bash-completion`

```
brew install bash-completion
```

添加 bash-completion 到`~/.bash_profile`:

```
 if [ -f $(brew --prefix)/etc/bash_completion ]; then
    . $(brew --prefix)/etc/bash_completion
 fi
```

❝`shell`有不同种类，我这里使用的是`bash`。

#### 代码没写完，突然要切换到别的分支怎么办？

暂存未提交的代码

```
git stash
```

还原暂存的代码

```
git stash apply
```

#### 怎么合并其他分支的指定Commit？

使用`cherry-pick`命令

```
git cherry-pick 指定commit-id
```

#### 本地临时代码不想提交，怎么一次性清空？

还原未`commit`的本地更改的代码

```
git reset --hard
```

还原包含`commit`的代码，到跟远程分支相同

```
git reset --hard origin/master
```

#### 已经提交的代码，不需要了，怎么当做没提交过？

还原到上次`commit`

```
git reset --hard HEAD^
```

还原到当前之前的几次`commit`

```
git reset --hard HEAD~2
```

强制推送到远程分支，确保没有其他人在`push`，不然可能会丢失代码

```
git push origin develop --force
```

#### 历史commit作者邮箱写错了，怎么一次性改过来？

使用`git filter-branch`命令。

复制下面的脚本，替换相关变量

* `OLD_EMAIL`
* `CORRECT_NAME`
* `CORRECT_EMAIL`

脚本如下：

```
#!/bin/sh

git filter-branch --env-filter '

OLD_EMAIL="your-old-email@example.com"
CORRECT_NAME="Your Correct Name"
CORRECT_EMAIL="your-correct-email@example.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
export GIT_COMMITTER_NAME="$CORRECT_NAME"
export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
export GIT_AUTHOR_NAME="$CORRECT_NAME"
export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

强制推送替换

```
git push --force --tags origin 'refs/heads/*'
```

#### 不小心把不该提交的文件commit了，怎么永久删除？

也是使用`git filter-branch`命令。

```
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch FILE-PATH-AND-NAME" \
  --prune-empty --tag-name-filter cat -- --all
```

强制推送覆盖远程分支。

```
git push origin --force --all
```

强制推送覆盖远程`tag`。

```
git push origin --force --tags
```

#### 怎么保证团队成员提交的代码都是可运行的？

这里想说的是使用`git hooks`，一般在项目目录`.git/hooks`，客户端可以使用`hooks`，控制团队`commit`提交规范，或者`push`之前，自动编译项目校验项目可运行。服务端可以使用`hooks`，控制push之后自动构建项目，`merge`等自动触发单元测试等。

#### `git reset --hard`命令，执行错了，能恢复吗？

查看当前`commit log`

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101811346-101811.png)

误操作`git reset --hard 8529cb7`

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101811598-101811.png)

执行`git reflog`

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101811718-101811.png)

还原到之前的样子

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101812182-101812.png)

#### 公司使用GitLab，平时还用GitHub，多账号SSH，如何配置？

编辑 `~/.ssh/config`文件 没有就创建

```
# github
Host github.com
Port 22
HostName github.com
PreferredAuthentications publickey
AddKeysToAgent yes
IdentityFile ~/.ssh/github_id_rsa
UseKeychain yes
User iisheng

# gitlab
Host gitlab.iisheng.cn
Port 22
HostName gitlab.iisheng.cn
PreferredAuthentications publickey
AddKeysToAgent yes
IdentityFile ~/.ssh/gitlab_id_rsa
UseKeychain yes
User iisheng
```

#### Git commits历史如何变得清爽起来？

多用`git rebase`。

比如，开发分支是`feature`，主干分支是`master`。我们在进行代码合并的时候，可以执行下面的命令。

```
# 切换当前分支到feature
git checkout feature

# 将当前分支代码变基为基于master
git rebase master
```

然后我们再切换到`master`分支，执行`git merge feature`，就可以进行快进式合并了，`commmits`历史就不会有交叉了。后文我们会详细讲解。

❝`git rebase`会更改commit历史，请谨慎使用。

下面的图是`Guava`项目的`commit`记录。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101812887-101812.png)

#### 如何修改已经提交的commit信息？

原始`Git`提交记录是这样的

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101813016-101813.png)

执行`git rebase -i 070943d`，对指定commitId之前的提交，进行修改

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101813735-101813.png)

修改后`Git`提交记录变成了这样

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101814053-101814.png)

❝`git rebase -i`非常实用，还可以将多个commit合并成一个等很多事情，务必要记下。

#### 不小心执行了`git stash clear`怎么办？

```
git fsck --lost-found
```

执行之后，可以找到相关丢失的`commit-id`，然后`merge`一下即可。

该命令上可以找回`git add`之后被弄丢的文件。

❝啥？你没执行过`git add`代码就丢了？别怕，一般编译器有`Local History`赶紧去试试吧。

##

## **详解git merge**

我们执行`git merge`命令的时候，经常会看到`Fast-forward`字样，`Fast-forward`到底是个什么东西？

其实，`git merge`一般有三种场景。

### 快进式合并

举个栗子，假如初始存在`master`和`hotfix`分支是这样的。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/14/640-20200814101814142-101814.jpg)

然后我们在`hotfix`分支加了些代码，分支变成这样了。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/14/640-20200814101814230-101814.jpg)

这个时候，我们将`hotfix`分支，`merge`到`master`，即执行`git merge hotfix`。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/14/640-20200814101814326-101814.jpg)

由于的分支`hotfix`所指向的提交`C3`是`C2`的直接后继， 因此`Git`会直接将指针向前移动。换句话说，如果顺着一个分支走下去能够到达另一个分支，那么`Git`在合并两者的时候， 只会简单的将指针向前推进（指针右移），因为这种情况下的合并操作没有需要解决的分歧——这就叫做 **快进（fast-forward）**。

### 三方合并

再举个栗子，假如初始存在`feature`和`master`分支情况是这样的。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101814530-101814.png)

然后我们在`feature`分支加了些代码，而`master`分支也有人加了代码，现在分支变成这样了。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101815248-101815.png)

这个时候，我们将`feature`分支，`merge`到`master`，即执行`git merge feature`。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101816065-101816.png)

和之前将分支指针向前推进所不同的是，`Git`将此次三方合并的结果做了一个**新的快照并且自动创建一个新的提交**指向它。这个被称作一次**合并提交**，它的特别之处在于他有不止一个父提交。

❝所以我们也知道了，为什么有的时候merge之后会产生新的commit，而有的时候没有。

### 遇到冲突时的合并

如果在两个分支分别对同一个文件做了改动，`Git`就没法直接合并他们。当遇到冲突的时候，`Git`会自动停下来，等待我们解决冲突。就像这样

```
$ git merge dev 
Auto-merging 111.txt
CONFLICT (content): Merge conflict in 111.txt
Automatic merge failed; fix conflicts and then commit the result.
```

我们可以在合并冲突后的任意时刻使用`git status`命令来查看那些因包含合并冲突而处于未合并`unmerged`状态的文件。

```
$ git status 
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
 both modified:   111.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

待解决冲突的文件`Git`会以未合并的状态标识出来，出现冲突的文件会出现一些特殊的区段，看起来像下面的样子。

```
<<<<<<< HEAD
111aaa
=======
111b
>>>>>>> dev
```

`<<<<<<<`后面的是当前分支的引用，我们的例子中，就代表`master`分支。`>>>>>>>`后面表示的是要合并到当前分支的分支，即`dev`分支。`=======`的上半部分，表示当前分支的代码。下半部分表示`dev`分支的代码。

我们可以把上面的测试内容改成下面的样子来解决冲突

```
111aaa
```

在解决了所有文件里的冲突之后，对每个文件使用`git add`命令来将其标记为冲突已解决。

解决冲突的过程中，每一步都可以执行`git status`查看当前状态，`Git`也会给出相应提示，进行下一步操作。当我们所有的文件都暂存之后时，执行`git status`时，`Git`会给我们看起来像下面的这种提示

```
$ git status 
On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)
```

然后，我们根据提示执行`git commit`。

```
Merge branch 'dev'

# Conflicts:
#       111.txt
#
# It looks like you may be committing a merge.
# If this is not correct, please remove the file
#       .git/MERGE_HEAD
# and try again.


# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
# All conflicts fixed but you are still merging.
#
```

然后，我们保存这次提交就完成了这次冲突合并。

##

## _**\\**_**详解git rebase\*\***

### rebase做了什么

举个栗子。我们同样用刚才`merge`的场景。

如果不用`rebase`，使用`merge`是下面这样的，合并分支的时候会产生一个合并提交，而且会有分支交叉的情况。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101816568-101816.png)

使用`rebase`是下面这样的。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101817763-101817.png)

然后，切换到`master`分支，进行一次快进式合并。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/14/640-20200814101819081-101819.png)

❝变基实际上就是基于其他分支重塑当前分支。变基之后，当前分支就相当于是基于最新的其他分支新加了一些commit，这样的话就可以进行快进式合并了。

### rebase原理

它的原理是首先找到这两个分支（即当前分支 `dev`、变基操作的目标基底分支`master`）的最近共同祖先 `C2`，然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件， 然后将当前分支指向目标基底`C3`, 最后以此将之前另存为临时文件的修改依序应用，也就是在`C3`后面添加`C4'`、`C5'`。

##

## _**\\**_**Git对象与快照\*\***

提到`Git`，总有人会说**快照**，**快照**是个什么鬼？

实际上，`Git`是一个内容寻址文件系统，其核心部分是一个简单的键值对数据库。将`Git`中的对象，存储在`.git/objects`目录下。

`Git`对象主要分为，**数据对象（blob object）**、**树对象（tree object）**、**提交对象（commit object）**、**标签对象（tag object）**。

### 数据对象

我们新建一个目录，然后在该目录下执行`git init`初始化一个`Git`项目。

然后，查看`.git/objects`目录下都有什么。

```
$ find .git/objects
.git/objects
.git/objects/pack
.git/objects/info
```

接着，我们写一个文件`echo '1111' > 111.txt`，并执行`git add`之后，再查看。

```
$ find .git/objects
.git/objects
.git/objects/5f
.git/objects/5f/2f16bfff90e6620509c0cf442e7a3586dad8fb
.git/objects/pack
.git/objects/info
```

我们发现`.git/objects`目录下，多了个文件和目录。实际上，`Git`会将我们的文件数据外加一个头部信息`header`一起做`SHA-1`校验运算而得到校验和。然后，校验和的前2个字符用于命名子目录，余下的38个字符则用作文件名。

我们可以使用下面的命令，显示在`Git`对象中存储的内容。

```
$ git cat-file -p 5f2f16bfff90e6620509c0cf442e7a3586dad8fb
1111
```

这就是我们在上文写入的文件内容。

上述类型的对象称之为**数据对象（blob object）**。数据对象，仅保存了文件内容，而文件名字没有被保存。

### 树对象

数据对象大致对应`UNIX`中的`inodes`或文件内容，树对象则对应了`UNIX`中的目录项。一个**树对象**包含了一条或多条**树对象记录（tree entry）**，每条记录含有一个指向数据对象或者子树对象的`SHA-1`指针，以及相应的模式、类型、文件名信息。

通常，`Git`根据某一时刻暂存区（即`index`区域）所表示的状态创建并记录一个对应的树对象。

当我们执行过`git add`之后，暂存区就有内容了，我们可以通过`Git`底层命令，生成树对象。

```
$ git write-tree
b716c7b049ccd9048b0566a57cfd516c17c1e39f
```

查看该树对象的内容。

```
$ git cat-file -p b716c7b049ccd9048b0566a57cfd516c17c1e39f
100644 blob 5f2f16bfff90e6620509c0cf442e7a3586dad8fb 111.txt
```

### 提交对象

数据对象保存了数据的内容，树对象可以表示当前目录的快照。但是，若想重用这些快照，必须记住树对象的`SHA-1`哈希值。而且，我们也不知道是谁保存了这些快照，在什么时刻保存的，以及为什么保存这些快照。而以上这些，正是**提交对象（commit object）**能保存的基本信息。

我们对当前暂存区进行一次提交，`git commit -m "first commit"`。

然后查看一下`log`找到该次提交的`commit`哈希值。

```
$ git log --oneline 
5281f7e (HEAD -> master) first commit
```

接着，我们查看一下该提交对象的内容。

```
$ git cat-file -p 5281f7e
tree b716c7b049ccd9048b0566a57cfd516c17c1e39f
author iisheng <***@gmail.com> 1596073568 +0800
committer iisheng <***@gmail.com> 1596073568 +0800

first commit
```

提交对象的格式很简单：它先指定一个顶层树对象，代表当前项目快照；然后是可能存在的父提交（前面描述的提交对象并不存在任何父提交）；之后是作者/提交者信息（依据你的`user.name`和`user.email`配置来设定，外加一个时间戳）；留空一行，最后是提交注释。

### 标签对象

**标签对象（tag object）** 非常类似于一个提交对象——它包含一个标签创建者信息、一个日期、一段注释信息，以及一个指针。主要的区别在于，**标签对象通常指向一个提交对象**，而不是一个树对象。它像是一个永不移动的分支引用——永远指向同一个提交对象，只不过给这个提交对象加上一个更友好的名字罢了。

❝实际上`Git`中的各种对象都是类似的，只不过因为各种对象自身功能不同，存储结构不同而已。

##

## **Git引用-我从远程拉的代码不是最新的？**

`Git`引用相当于是`Git`中特定哈希值的别名。一长串的哈希值不是很友好，但是起个别名，我们就可以像这样`git show master`、`git log master`的去使用他们。

`Git`中的引用存储在`.git/refs`目录下。我们可以执行`find .git/refs/`查看当前`Git`项目中都存在哪些引用。

### HEAD引用

在`.git`目录下有一个名字叫做`HEAD`的文件，`HEAD`文件通常是一个符号引用（symbolic reference）指向目前所在的分支。所谓符号引用，表示它是一个指向其他引用的指针。

如果我们在工作区`checkout`一个`SHA-1`值，`HEAD`引用也会指向这个包含`Git`对象的`SHA-1`值。

### 标签引用

`Git`标签分为，附注标签和轻量标签。轻量标签，使用`git tag v1.0`即可创建。附注标签需要使用`-a`选项，即`git tag -a v1.0 -m "my version 1.0"`这种。

轻量标签就是一个固定的引用。附注标签需要创建标签对象，并记录一个引用来指向该标签对象。

### 远程引用

不熟悉`Git`的同学，可能会犯这样一个错误。其他同学让他拉取一下远程最新的`master`分支代码，他可能直接用`IDE`找到本地的远程分支的引用，也就是`origin/master`，直接`checkout`一个本地分支。

其实，`origin/master`只是远程分支的一个引用，不一定跟远程分支代码同步，我们可以用`git fetch`或者`git pull`来让`origin/master`和远程分支同步。

参考文献：

\[1]: [https://git-scm.com/](https://git-scm.com/)
