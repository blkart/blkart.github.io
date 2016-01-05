---
layout: post
title: pull request方式向GitHub上的项目贡献代码『1』
date: 2015-03-02
categories: blog
tags: [GitHub]
description: pull request方式向GitHub上的项目贡献代码

---

#### 最近用到了pull request方式向GitHub上的项目贡献代码，总结下具体步骤：

## 第一次贡献代码

当我们需要向github上的项目提交代码时，我们的代码并不能直接push到上游项目的代码仓库中，这时我们的代码要放到哪呢？放到我们自己github帐号的项目代码仓库中。这个代码仓库并不是我们自己创建的，而是将上游项目的代码仓库fork到我们的github帐号中。然后将我们的代码push到我们自己的仓库中，最后创建pull request，等待上游项目的管理者review/merge。

### 具体步骤

#### 访问上游项目的GitHub代码仓库，将上游项目fork到自己的github帐号 

  ![选区_080](/img/blog_img/080.png) 
  ![选区_081](/img/blog_img/081.png) 

#### 在fork到的代码仓库中基于master branch创建一个新的branch，用于存放我们要提交的代码 

  ![选区_082](/img/blog_img/082.png) 

#### 将自己帐号中的项目仓库clone到本地

    # git clone https://github.com/blkart/test-repo.git

#### 访问到本地项目仓库目录

    # cd test-repo
    # git branch
    * master

#### 切换到前面步骤中新建的branch

    # git branch -r
    * master
      new-arch
    # git checkout arch-new
    # git branch
      master
    * new-arch

#### 修改或添加代码

#### 提交修改，并push到GitHub，这样我们对代码的修改就提交到自己帐号的代码仓库中了

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
