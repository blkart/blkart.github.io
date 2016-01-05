---
layout: post
title: openstack glance image 制作流程
date: 2015-07-07
categories: blog
tags: [openstack,glance]
description: openstack glance image 制作流程

---

### 概述

OpenStack环境中可以基于Glance Image快速创建云主机，所使用到的Glance Image可以从互联网上下载([常见linux发行版本openstack image下载地址汇总](http://blog.blkart.org/blog/2015/07/07/openstack-glance-image-download/))。但是当我们希望基于某个Glance Image创建的云主机中提前预装一些自定义的软件包时，我们就需要对Glance Image进行二次制作。

### 写在前面

现在网上可以找到一些 OpenStack Glance Image 制作的文档/教程。有些文档/教程中所描述的方法制作出的 Image 虽然可以在 OpenStack 环境中使用，但是并不能与 OpenStack 环境紧密结合（如 OpenStack 无法获取 Instance 启动日志等）。因此不建议根据网上的文档/教程自己制作 Image 。然而，各 Linux 发行版本官方提供的 Glance Image 已经对镜像进行过对应的处理，可与 OpenStack 环境紧密结合。因此，**强烈建议根据本文档描述的方法对 Linux 发行厂商官方提供的 Glance Image 进行二次定制**。

### 准备环境

准备一台安装了 CentOS 7 操作系统的服务器，并且安装以下工具包。

    # yum -y install libvirt qemu-kvm-rhev libguestfs-tools

### 制作流程

下面以在 CentOS 7.0 中预装 MySQL 数据库为例，演示下 Glance Image 的二次制作流程。

#### 下载 Image

从 CentOS 官方下载 CentOS 7.0 系统的 Image 到本地，[下载地址](http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2)。

    # wget http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2

#### 设置 Image root 用户密码

前面一步下载的 Image 中 root 用户默认密码被锁定，后续步骤中我们需要以 root 用户登录到系统中对系统进行定制，因此需要设置 root 用户密码。

    # virt-sysprep --root-password password:abc123 -a CentOS-7-x86_64-GenericCloud.qcow2

> **注意：** 此处 --root-password password:eayun123 中 "abc123" 为所设置的 root 用户的密码。

#### 启动虚拟机

基于前面所下载的 Image 作为虚拟机磁盘，启动一个虚拟机。

    # qemu-kvm -vnc 0.0.0.0:20 -m 2048 -hda CentOS-7-x86_64-GenericCloud.qcow2

#### 定制系统

通过VNC方式连接前面启动的虚拟机，以 root 用户身份登录到系统中，对系统进行定制（如安装自定义软件包）。定制完成后关闭虚拟机。

    # yum -y install mariadb mariadb-server

> **警告：** 定制化操作完成后，需要删除并锁定 root 用户密码

    # password -d root
    # password -l root

操作完成，关机

    # poweroff

#### Image 初始化

使用 virt-sysprep 命令对 Image 进行初始化，初始化操作会将 Image 中一些唯一性信息进行清除（如SSH host keys、网卡MAC地址等），具体清除内容可以查看 virt-sysprep 帮助手册。

    # virt-sysprep -a CentOS-7-x86_64-GenericCloud.qcow2

#### 消除 Image 空洞

qcow2 格式的 Image 有稀疏的问题，在磁盘级别上看，镜像会有大量连续的相同空洞，通过virt-sparsify 我们可以消除这些空洞，从而缩小磁盘大小，便于我们传输磁盘镜像。

    # virt-sparsify --compress CentOS-7-x86_64-GenericCloud.qcow2 CentOS-7.0-x86_64-MySQL.img

#### 测试 Image

将二次定制的 Image 上传到 OpenStack 环境中进行测试，测试没有问题， Image 二次定制完成。
