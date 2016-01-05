---
layout: post
title: pull request方式向GitHub上的项目贡献代码『2』
date: 2015-03-03
categories: blog
tags: [GitHub]
description: pull request方式向GitHub上的项目贡献代码

---

上一篇介绍了第一次贡献代码时的操作方法，这篇来了解下第二次及后续贡献代码的方法。

## 再次贡献代码

第一次贡献代码后，在我们自己的GitHub帐号中就有了一个从上游fork的项目。如果需要再次向上游贡献代码，我们需要考虑一个问题：从我们第一贡献代码到现在，上游项目的master分支是否有了新的commit，如果有，我们需要先将本地master分支与上游master分支进行同步，然后在我们要修改的分支中rebase，最后修改代码。否则上游项目管理者在merge我们的pull request时可能会出现冲突。

### 具体步骤

#### 新建一个remote 

> **_upstream_**为新remote名，可自定义

    # git remote add upstream https://github.com/XXX/YYY.git

#### 查看所有remote，此时应该可以看到以**_upstream_**开头的信息

    # git remote -v
    ......
    upstream    https://github.com/XXX/YYY.git (fetch)
    upstream    https://github.com/XXX/YYY.git (push)

#### 将新remote对应的项目下载到本地

    # git fetch upstream

#### 将上游master合并到本地master

    # git checkout master
    # git merge upstream/master

#### 查看git log，此时应该可以看到上游master分支最新的一个commit

    # git log

#### 将更新push到GitHub（到此完成了fork下来的项目与上游项目master分支的同步）

    # git push

#### 更新new-arch分支

    # git checkout new-arch
    # git rebase master

#### 修改代码

#### 提交修改，并push到github

    # git add *
    # git commit -s
    # git push

#### 访问GitHub上自己帐号的代码仓库，创建pull request，将修改提交到上游

> **注意**
>
> 第三个图中**_base fork_**及**_base_**设置的是上游项目及分支，**_head fork_**及**_compare_**设置的是本地项目及分支），图示为将new-arch分支提交到上游项目的master分支。

![选区_083](/img/blog_img/083.png) 
![选区_084](/img/blog_img/084.png) 
![选区_086](/img/blog_img/086.png) 

#### 等待上游项目管理者review/merge
