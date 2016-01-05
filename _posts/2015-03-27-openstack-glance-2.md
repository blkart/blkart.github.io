---
layout: post
title: Glance组件之深入探索『2』——Image相关
date: 2015-03-27
categories: blog
tags: [openstack,glance]
description: Glance组件之深入探索

---

前面一篇blog对Glance组件做了简单介绍，其中提到了Glance组件为OpenStack提供Image Service，这篇我们来了解下Glance是如何实现Image管理的。

#### Image是如何存放的？

Image的元数据(metadata)通过glance-registry(V1)存放在数据库中； image的数据(data)通过glance-store存放在各种backend store(如eqlx、ceph等)中。

#### Image的访问权限有哪些？

* public 公共的(is_public=true)：可以被所有的tenant使用。

* private 私有的/项目的(is_public=false)：只能被image owner所在的tenant使用。

* shared 共享的(is_public=false)：一个非公共的image可以共享给另外的tenant，可通过member操作来实现(后续会单独介绍)。

* protected 受保护的(protected=true/false)： protected=true的image不能被删除。

#### Image的的状态有哪些？

* queued：只在数据库中存放了元数据(metadata)，未上传Image data。

* saving：正在上传Image data。

* active：正常状态。

* deleted : 已删除。

* pending_delete： 等待删除。

* killed：image 元数据不正确，等待被删除。

#### Image状态变化机制：

![选区_094](/img/blog_img/094.png)
