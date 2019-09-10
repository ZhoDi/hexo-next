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


## 创建仓库
```bash
git init
```

{% codeblock lang:csharp %}
hexo.on('generateBefore', function () {
  if (hexo.locals.get) {
    var data = hexo.locals.get('data');
    if ( data && data.next ) {
      if ( data.next.override ) {
        hexo.theme.config = data.next;
      } else {
        merge(hexo.theme.config, data.next);
      }
    }
  }
});
{% endcodeblock %}

## 查看版本库状态
```bash
git status
```

## 查看历史以及版本回退
```bash
# 详细信息
git log
# 单行信息
git log --pretty=oneline
1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master) append GPL
e475afc93c209a690c39c13a46716e8fa000c366 add distributed
eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0 wrote a readme file

# 首先，Git必须知道当前版本是哪个版本,在Git中，用HEAD表示当前版本，也就是最新的提交1094adb...
# 上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100

# 使用下面这个命令便退回了上个版本,但是假如我们觉得之前的代码又需要了,我们还能把它找回来吗?
git reset --hard HEAD^

# 当然是可以的,Git提供了一个命令git reflog用来记录你的每一次命令：
git reflog
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
e475afc HEAD@{3}: commit: add distributed
eaadf4e HEAD@{4}: commit (initial): wrote a readme file

# 然后使用版本回退命令，这样我们又回到了最初的版本
git reset --hard 1094a
```

## 添加、修改、删除文件
```bash
git add 修改的文件.txt
git add 新增的文件.txt 新增的文件2.txt
git add/rm 删除的文件.txt  #add和rm都可以
# 不用手动删除再add到暂存区,也可以直接git rm 删除的文件.txt(这样相当于先手动删除,再git add/rm 删除的文件.txt)

# 可以多次使用git add命令将修改的文件或新增的文件添加到暂存区
git commit -m "新增了三个txt文件"
# -m 表示本次提交备注的信息
```
1. 我们本地是同时存在工作区和版本库的,版本库中又包含stage( 缓存区 )和仓库分支(下面用master代替) **如下图**

2. 我们把文件往Git版本库里添加的时候，是分两步执行的：

3. 第一步是用git add把文件添加进去，实际上就是把文件修改添加到暂存区；

4. 第二步是用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支。
因为我们创建Git版本库时，Git自动为我们创建了唯一一个master分支，所以，现在，git commit就是往master分支上提交更改。

5. 你可以简单理解为，需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。


{% asset_img 0.jpg 工作区暂存区图解 %}

{% note info %}
  <p style="font-size: 20px;">拓展</p>
  `git diff` 是工作区和暂存区进行比较, `git diff --cached` 是暂存区和master进行比较
{% endnote %}

1. 一开始没有做修改时,三个地方的代码全部相同

2. 修改后工作区发生变化,使用 `git diff` 可以检测出工作区和缓存区的不同, 而 `git diff --cached` 什么都检测不到,因为我们上面说过 `git add` 是将修改添加到了缓存区,而我们现在只是修改了工作区,并没有使用 `git add` 所以暂存区和master目前还是相同的

3. 然后我们使用 'git add 修改的文件.txt' 这时工作区修改同步到暂存区 `git diff` 什么都检测不到,而`git diff --cached` 反而可以检测到了

4. 撤销修改(新增和删除也是一种修改)

```bash
# 第一种情况 我们只进行了修改还没有使用git add那么可以直接使用下面的命令丢弃工作区修改就OK
git checkout -- readme.txt
# 第二种情况 我们不仅修改了还git add到了暂存区如何办呢?先使用下面的命令丢弃暂存区修改,再用第一条命令丢弃工作区修改就OK
git reset HEAD readme.txt
```