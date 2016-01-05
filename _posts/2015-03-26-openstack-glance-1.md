---
layout: post
title: Glance组件之深入探索『1』——简介
date: 2015-03-26
categories: blog
tags: [openstack,glance]
description: Glance组件之深入探索

---

最近在研究OpenStack的Glance组件，在这里做一个简单整理。Glance组件为OpenStack提供Image Service，负责镜像的上传，下载等管理。Glance跟其他组件（如nova）相比，功能比较简单，分析起来更容易一些。

Glance组件通过REST API提供以下对镜像的操作：

* 查询
* 注册
* 上传
* 获取
* 删除
* 访问权限管理

本次看的OpenStack版本（Juno）里，Glance组件同时支持两个版本的REST API：V1和V2。两个版本区别如下：

#### V1：

* 由两个服务支撑：glance-api、glance-registry(提供对数据库的操作)；
* 只提供了基本的image和member操作功能：镜像创建、删除、下载、列表、详细信息查询、更新，以及镜像member成员的创建、删除和列表。

#### V2：

* 只需要一个服务支撑：glance-api，该版本将V1版本的glance-registry服务的所提供的功能整合到了glance-api中；
* 除了支持V1的所有功能外，主要是增加了如下功能：镜像 location 的添加、删除和修改等操作、metadata namespace 操作、image tag 操作。

> **_注：_**两个版本对 image store 的支持是相同的。

参见下图： 

![选区_090](/img/blog_img/090.png)

目前，Horizon和Glance CLI默认使用的是V1版本的API，网上有些资料说V2的性能差一些(需要再研究下)，另外V1版本所提供的功能已经可以满足镜像管理的必备功能。关于Horizon及Glance CLI如何使用V2版本的API，后续单独介绍。

前面提到了两个服务，具体介绍下：

* **_glance-api_** ： 该服务主要负责接收响应镜像管理命令的Restful请求，分析消息请求信息并分发其所带的命令（如新增，删除，更新等）。V1版本中涉及到数据库的操作会调用registry-clinet生成响应的http命令转发给glance-registry服务。默认绑定端口是9292。

* **_glance-registry_** ： 该服务主要负责接收响应镜像元数据命令的Restful请求。分析消息请求信息并分发其所带的命令（如获取元数据，更新元数据等），进行数据库相关的增删改查工作。V2版本中已将该服务整合到glance-api中。默认绑定的端口是9191。

> **_注：_**Glance组件通过9292端口向外提供服务，9191端口只被glance-api调用。
