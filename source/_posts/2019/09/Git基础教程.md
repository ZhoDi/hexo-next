---
title: Git基础教程
date: 2019-09-10 17:53:07
updated: 2019-09-10 17:53:07
comments: true
tags: 
  - git
---

<blockquote class="blockquote-center">Git基础入门, 主要介绍一些常用命令</blockquote>
<!--more-->

{% note info %}
# 认识Git 
{% endnote %}

Git是一个开源的分布式的版本控制系统。它可以追踪任何变化的文件，支持完整的工作流程，来保证数据的完整性和处理事务的高效性。
可他的作用到底在哪呢?
当我们进行开发的时候,每时每刻都在对文件进行修改。但是当我们发现问题想要回退怎么办呢?难道每次修改,把文件复制一份保存下来?
这样虽然可以,但时间一长你就会发现越来越多越来越难以管理。而且我们一般都是多人开发,难道我改了一个文件还要把东西发给同事?
Git的作用就在这里了,它能够帮你管理,你只需要专注于工作。


## Git工作流程

{% asset_img Git工作流程.png 1-1_工作流程 %}

几个专用名词的译名如下

- Workspace：工作区
- Index / Stage：暂存区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库

图上一共有六个箭头,代表着六个常用的命令,下面就来介绍一下它们
- 首先通过`git clone [远端仓库地址]`克隆一个仓库到本地
- 对文件进行修改,修改完成之后`git add [文件名]`到暂存区(并不需要把所有修改的文件都添加到暂存区)
- 确认无误之后`git commit -m '本次提交描述'`到仓库中(只会提交你添加到暂存区的修改)
- 当发现自己写的代码有bug,而且这个时候还没有推送到远端,那么就用`git checkout -- [文件名]`撤销修改
- 没有发现bug,在通过`git push [远端名] [本地分支名]`推送到远端
- 这个时候同事通过`git pull`拉取你推送的代码到本地,这样就完成了一整个流程

## 关于 暂存区 工作区 版本库

1. 我们本地是同时存在工作区和版本库的,版本库中又包含stage( 暂存区 )和仓库分支(下面用master代替) **如下图**

2. 我们把文件往Git版本库里添加的时候，是分两步执行的：

3. 第一步是用git add把文件添加进去，实际上就是把文件修改添加到暂存区；

4. 第二步是用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支。
因为我们创建Git版本库时，Git自动为我们创建了唯一一个master分支，所以，现在，git commit就是往master分支上提交更改。

5. 你可以简单理解为，需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。

{% asset_img 0.jpg 工作区暂存区图解 %}

`git diff` 是工作区和暂存区进行比较, `git diff --cached` 是暂存区和master进行比较

1. 一开始没有做修改时,三个地方的代码全部相同

2. 修改后工作区发生变化,使用 `git diff` 可以检测出工作区和暂存区的不同, 而 `git diff --cached` 什么都检测不到,因为我们上面说过 `git add` 是将修改添加到了暂存区,而我们现在只是修改了工作区,并没有使用 `git add` 所以暂存区和master目前还是相同的

3. 然后我们使用 'git add 修改的文件.txt' 这时工作区修改同步到暂存区 `git diff` 什么都检测不到,而`git diff --cached` 反而可以检测到了




{% note info %}
# 常用命令
{% endnote %}

## 创建仓库

```bash
# 将当前目录变成一个Git仓库,生成一个.git文件夹,这个文件夹就是git用来管理仓库的,不要修改它
git init

# 将远端与本地创建的仓库关联起来
# 当我们第一次关联,而且本地有过提交,这样我们是推送不上去的,这时我们就要加上--allow-unrelated-history参数
git remote add origin [远端仓库地址]

# 克隆远端仓库
git clone [远端仓库地址]
```
## 配置GitHub

我们通过`ssh`地址克隆远端仓库的时候会发现没有权限

- 修改本地git账户信息
 `git config --global user.name "[用户名]"`
 `git config --global user.email "[邮件地址]"`

- 打开本地命令行

- 执行`ssh-keygen -t rsa -b 4096 -C "邮件地址"`

- 点三次回车,这样会把秘钥保存在默认位置

- 找到秘钥`C:\Users\zhaodi\.ssh\id_rsa.pub`用记事本打开复制

- 找不到的执行`clip < ~/.ssh/id_rsa.pub`命令直接复制到剪贴板

- 复制到[New SSH key](https://github.com/settings/keys) 详见下一步

- 打开github登录 > 点击右上角个人头像 > settings > SSH and GPG keys > New SSH key > 填写Title和Key > Add SSH key


## 修改、提交、推送 (新增和删除对存储库来说也是一种修改)

```bash
# 将文件添加到暂存区,无论这个文件是本身存在的,还是新增的
git add 修改的文件.txt
git add 新增的文件1.txt 新增的文件2.txt

# rm相当于先手动删除,再git add 删除的文件.txt
# 但是我们也可以加上--cached将它从版本库中移除,但是仍保留在磁盘中 (相当于在下载器中删除任务,但是不删除文件)
git rm 删除的文件.txt

# 将暂存区的所有修改提交到本地版本库中, -m 表示本次提交备注的信息
git commit -m "新增了三个txt文件"

# 推送到远端 -u参数 将本地的master分支推送到origin主机，同时指定origin为默认主机，后面就可以不加任何参数使用git push了
git push -u origin master

# 指定关联远端分支
git branch --set-upstream-to=origin/developer developer
```

## 撤销修改和版本回退
```bash
# 撤销修改
# 第一种情况 我们只进行了修改还没有使用git add那么可以直接使用下面的命令丢弃工作区修改就OK
git checkout -- readme.txt
# 第二种情况 我们不仅修改了还git add到了暂存区如何办呢?先使用下面的命令丢弃暂存区修改,再用第一条命令丢弃工作区修改就OK
git reset HEAD readme.txt

# 版本回退
# 首先，Git必须知道当前版本是哪个版本,在Git中，用HEAD表示当前版本，也就是最新的提交1094adb...
# 上一个版本就是HEAD^，上上一个版本就是HEAD^^，上十个版本就可以直接加数字HEAD~10
# 使用下面这个命令便退回了上个版本,--hard参数会重置工作区和暂存区
git reset --hard HEAD^

# 我们回退代码之后后悔了怎么办?,Git提供了一个命令git reflog用来记录你的每一次命令：
git reflog
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: 修改

# 然后使用版本回退命令，这样我们又回到了最初的版本
git reset --hard 1094a
```

## 分支管理

```bash
# 创建一个分支
git branch developer

# 切换到创建的分支  不建议用旧版的checkout,容易和撤销修改混淆
git switch developer (git checkout developer)

# 我们也可以加上 -b 参数创建并切换
git switch -c developer (git checkout -b developer)

# 列出所有分支,当前分支前会加上*号
git branch

# 合并分支到当前分支,加--no-ff参数代表禁用Fast forward模式,可以看到合并记录
git merge developer

# 删除分支
git branch -d developer

# 查看远端仓库,可以看到关联信息等
git remote show origin

# 指定关联远端分支
git branch --set-upstream-to=origin/developer developer
```

## 查看状态以及历史
```bash
# 详细信息
git log
# 单行信息
git log --pretty=oneline

git status    #完整信息
git status -s #简略信息
git diff      #显然这些信息太过于模糊,所以还有`git diff`帮助你查看详细信息
```
