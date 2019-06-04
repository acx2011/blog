---
layout: post
title: Git 使用手册
tags: [Dev-Tool]
---

**摘要：**这篇文章是自己在学习和使用 Git 过程中的总结，希望能通过划重点的方式把 Git 相关的知识点和常用的命令进行汇总，只要一篇文章就能够轻轻松松上手和方方便便查询。
{:.message}

## 写在前面

其实对于像我这样的普通程序猿来讲，Git 和 Svn 一样，都是在开发中不得不使用的版本控制软件，或者说是工具。所以我也把这篇文章归在了 [Dev-Tool](/blog/tag/dev-tool/) 这个标签里。既然是工具，我认为就没有必要把时间浪费在学习那些 “高冷” 的命令上。等到非用不可了，也可自行 Google 或查阅 [Git 官方文档](https://git-scm.com/book/zh/v2)。

## 安装 Git

本来想像模像样地先来段 **Git 介绍** 的，但又一想还是别复制粘贴了。既然标榜的是知识点汇总，就直接开门见山吧。先贴一个 [Git 下载地址](https://git-scm.com/downloads)。

以 Windows 为例，安装完会产生一个 `git-bash.exe` 的命令行程序，使用这个命令行程序就可以像在 Linux 中敲命令式地使用 Git 了。这个命令行其实和 Windows 自带的 `cmd.exe` 差不多，但是有两点我认为比 `cmd.exe` 要好一些：**一是主题颜色更有逼格，二是可以直接使用 Linux 命令。**

比如，在 `cmd.exe` 中进入 `D` 盘中的一个文件夹得先使用 `d:` 命令才能进行 `cd` 和 `dir` 等命令。但是在这个 `Git Bash` 中就可以直接 `cd /d` 然后 `ll` 了。对于不想学 `cmd` 命令只会 Linux 命令的同学四不四 hin 方便。不过最近我都用 [Xshell](http://www.netsarang.com/download/down_form.html?code=522) 当开发中的命令行工具了，不管在 Windows 还是 Linux 下，我不会告诉你这是因为 `Xshell` 可以添加很多常用命令的快捷按钮的！简直就是我等懒手党的福音。

## 必要设置

#### 设置用户名和邮箱

到此 Git 环境就算基本搭建好了。不过还得先在命令行中输入以下命令进行全局设置：

```shell
git config --global user.name "your_name"
git config --global user.email "your_email@example.com"
```

这两行命令之所以有必要设置，一是因为在团队开发中自然需要体现 “我” 是谁，二是即便用 Git 仅仅作为 [Github](https://github.com/) 等其他代码托管网站的工具，不把全局的 `user.email` 设置成 [Github --> Personal settings --> Emails](https://github.com/settings/emails) 中对应的邮箱地址，Github 就无法把每次本地的提交算入 `contributions`。这个坑我是踩过的，提交了近一个月的代码后突然发现 `contributions` 居然没有增加，直到仔细阅读了 [Learn how we count contributions](https://help.github.com/articles/why-are-my-contributions-not-showing-up-on-my-profile/)。#围笑脸#

#### 生成 SSH Key

说到使用 Git 作为 Github 的工具了，还有一个必要的设置，那就是需要在本地生成一份 `SSH Key`。因为本地 Git 版本库和 Github 远程仓库之间的传输是通过 `SSH` 加密的，只有创建了 `SSH Key`，才可以把本地 Git 版本库的修改推送到 Github 的远程仓库。

```shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

使用上面这行命令并替换自己的邮箱地址（**不一定非要和全局设置中的邮箱地址一致**），然后一路回车使用默认值即可。如果一切顺利的话，便可以在用户主目录下的 `.ssh` 目录里找到 `id_rsa` 和 `id_rsa.pub` 两个文件。这两个就是 `SSH Key` 的秘钥对：`id_rsa` 是私钥，不能泄露出去；`id_rsa.pub` 是公钥，可以放心地告诉任何人。最后需要把 `id_rsa.pub` 里的内容填入 [Github --> Personal settings --> SSH keys](https://github.com/settings/keys) 中 "New SSH Key" 的页面里并给这个 `SSH Key` 起一个 "title"，比如说 "Company"、"Home" 等等类似可以轻易区分出是哪台电脑的 `SSH Key` 的名称。这样，凡是在 Github 中保存了对应 `SSH Key` 的机器就可以向 Github 的远程仓库推送修改了。

#### 解决连接超时的问题

在我们测试 Git 是否成功连接 Github 时，使用 `ssh -T git@github.com`。如果出现：`You've successfully authenticated`，那么恭喜你，连接成功可以使用了。如果出现：`ssh: connect to host github.com port 22: Connection timed out`，很遗憾，连接超时。

解决方法是：进入 Git 的安装目录，找到 `/etc/ssh/ssh_config` 文件，把如下内容复制到该文件的末尾处并保存：

```
Host github.com
User git
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```

## 几个概念

使用 Git 的大体流程是：在本地的工作区进行开发和修改，提交时先把修改放入暂存区，再把暂存区的所有修改提交到本地的版本库，最后把本地的版本库推送至类似于 Github 这样的远程仓库当中。

这样可以看出，Git 这种分布式版本系统的最大好处之一是在本地工作完全不需要考虑远程库的存在，也就是有没有联网都可以正常工作，当有网络的时候，再把本地提交推送一下就完成了同步。而 Svn 在没有联网的时候是没法干活的。

* **工作区**：电脑里能看到的文件和目录，或者说就是物理上的项目代码文件。可类比为我们生活中要存的现金。

* **暂存区**：虚拟的一块区域，需要把想要提交的修改先放入其中。可类比为银行柜台的业务员，我们需要把要存的钱都先交给这个业务员。

* **本地库**：每一个工作区都有一个隐藏的目录 `.git`，这个就是该项目仓库的本地库。可类比为当地银行的个人钱库，银行的业务员会把每天客户交存的所有钱都放入这个个人钱库中。

* **远程库**：类似于 Github 这样的远程代码托管仓库。可类比为个人的银行账户，银行账户只接受个人钱库里存入现金的金额。

![Git 流程](/blog/assets/img/docs/Git/01.jpg)

## 命令大全

好了，下面就全是各种命令和介绍了。

#### `git init`
在一个目录下创建项目仓库，同时 `Git `自动为该仓库创建了唯一一个 `master` 分支。一个目录只有成为项目仓库后，这个目录里的所有文件才可以被 Git 管理起来，每个文件的修改记录都能够被跟踪。不过使一个目录成为项目仓库的途径不仅仅是通过 `init` 命令，还可以通过下面的 `clone` 命令直接下载一个项目仓库。不管通过什么方式形成的项目仓库，其目录下一定会有一个隐藏的 `.git` 目录，这个目录是 Git 用来跟踪管理该代码仓库的，没事不要随意修改这个目录里面的文件，不然该仓库可能会被破坏。

#### `git clone <远程仓库地址>`
这个命令的使用场景是：先在 Github 上创建好了一个远程仓库，然后直接把远程库下载到本地成为本地库，本地并不需要使用 `git init` 创建仓库。**最好的方式是就先创建远程库，然后从远程库克隆。**因为如果有多个人协作开发，那么每个人各自从远程克隆一份就可以了。实际上，Git 也支持 `https` 等其他多种协议，默认的 `git://` 使用 `ssh`。使用 `https` 除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放 `http` 端口的公司内部就无法使用 `ssh` 协议而只能用 `https`。

#### `git clone <远程仓库地址> -b <分支名> <文件夹名>`
**建立分支时，推荐使用这种方法先在远程建立一个开发分支，然后直接 `clone` 该分支。**这样做的好处是不用在一个项目中来回切换 `master` 分支和开发分支。

#### `git remote add origin <远程仓库地址>`
这个命令的使用场景是：分别在 Github 上和本地都创建了项目仓库，需要把两者进行关联。

#### `git remote -v`
查看远程库的详细信息。远程仓库的默认名称是 `origin`。如果没有推送权限，就看不到 `push` 的地址。

---

#### `git add <文件名>`
把文件添加到仓库。**实际上是把工作区的文件修改添加到暂存区。** `git add .*` 将所有修改的文件添加到仓库。

#### `git commit -m "提交说明"`
把所有添加到仓库的文件一次性提交到分支。**实际上是把暂存区的所有内容提交到当前分支。**提交修改和提交新文件都需要先 `add` 然后再 `commit`。可以这样理解：把所有要提交的文件修改先放到暂存区，然后一次性提交暂存区里的所有修改。每次修改如果不先 `add` 到暂存区，那就不会加入到 `commit` 中。由此可见：**Git 跟踪并管理的是修改，而不是文件。**

#### `git status`
查看仓库当前的状态。修改后或提交前都可以先通过该命令随时掌握工作区的状态。如果该命令显示有文件被修改过，可以用下面的 `diff` 命令查看修改的内容。

#### `git diff <文件名>`
查看一个文件的修改内容。

#### `git diff HEAD --<文件名>`
查看一个文件在工作区和版本库中的最新版本的区别。

#### `git log`
显示从最近到最远的提交历史记录。加上 `--pretty=oneline` 参数呈简洁模式，第一列为每次提交的版本号 `commit_id`。`--graph` 参数可以看到分支合并图。

#### `git reflog`
查看每次使用命令的记录。有过提交记录的也可以看到 `commit_id`。

#### `git reset --hard <版本号或commit_id>`
添加了 `commit_id` 是回滚到指定版本，不添加 `commit_id` 就是把工作区的所有修改回滚。在 Git 中，用 `HEAD` 指针指向当前版本，上一个版本号是 `HEAD^`，上上一个版本号是 `HEAD^^`，第 100 个版本号是 `HEAD~100`。

#### `git reset HEAD <文件名>`
撤销暂存区的修改，重新放回工作区。

#### `git checkout --<文件名>`
撤销工作区的修改甚至删除，让文件回到最近一次 `git commit` 或 `git add` 时的状态。这里有两种情况，一种是文件自修改后还没有被放到暂存区，现在撤销修改就回到和版本库一模一样的状态；一种是文件已经添加到暂存区后又作了修改，现在撤销修改就回到添加到暂存区后的状态。其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

#### `git rm <文件名>`
删除一个文件。如果一个文件已经被提交到版本库，就不用担心误删。不过要小心，只能恢复文件到最新版本，但是会丢失最近一次提交后修改的内容。

---

#### `git pull`
在当前分支上拉取远程对应分支的新提交。

#### `git push origin <分支名>`
本地作了提交后把本地库分支推送到远程仓库。如果推送失败，先用 `git pull` 拉取远程的新提交。当从远程仓库克隆时，实际上 Git 自动把本地的 `master` 分支和远程的 `master` 分支对应起来了。推送成功后，可以立刻在 Github 页面中看到远程库的内容和本地是一模一样的。

#### `git checkout -b <分支名>`  = `git branch <分支名>` + `git checkout <分支名>`
创建新分支，然后切换到新分支。本地新建的分支如果不推送到远程，对其他人就是不可见的。

#### `git checkout -b <分支名> origin/<分支名>`
在本地创建和远程分支对应的分支，分支名最好一致。

#### `git branch --set-upstream <分支名> origin/<分支名>`
建立本地分支和远程分支的关联。

#### `git branch`
列出本地库中的所有分支，当前分支前面会标一个 `*` 号。加上 `-a` 参数可以看到远程库中的其他分支。

#### `git merge --no-commit --no-ff <分支名>`
合并指定分支到当前分支。加上 `--no-commit` 避免自动提交，同时可以看到 merge 的内容，而且手动提交是可以添加提交的注释。加上 `--no-ff` 参数时会生成一个新的 `commit`，这样从分支历史上就可以看出来曾经做过合并。

#### `git branch -d <分支名>`
删除分支。如果要删除一个没有被合并过的分支，可以通过 `git branch -D <分支名>` 强行删除。

#### `git push --delete origin <分支名>`
删除远程分支。如果提示 `remote ref does not exist`，使用 `git fetch -p origin` 清除本地缓存。

---

#### `git tag -a <标签名> -m "标签说明" <commit_id>`
给分支打标签，后面加上 `commit_id` 就是给 `commit` 打标签，`-a` 参数指定标签名，`-m` 参数指定说明文字

#### `git tag`
查看所有标签。

#### `git show <标签名>`
查看标签说明。

#### `git tag -d <标签名>`
删除一个标签。创建的标签都只存储在本地，不会自动推送到远程。所以打错的标签可以在本地安全删除。

#### `git push origin <标签名>`
推送标签到远程仓库。

#### `git push origin --tags`
推送全部未推送过的本地标签到远程仓库。

--- 

#### 重置远程的 master 分支

有时 master 分支因为某次错误的提交，即使回滚了也还有很多相应的无用的历史记录。如果想清除这些历史记录的话，那么可以使用以下命令重置远程的 master 分支：

```sh
## 1. Checkout
git checkout --orphan latest_branch

## 2. Add all the files
git add -A

## 3. Commit the changes
git commit -am "commit message"

## 4. Delete the branch
git branch -D master

## 5.Rename the current branch to master
git branch -m master

## 6.Force update repository
git push -f origin master
```