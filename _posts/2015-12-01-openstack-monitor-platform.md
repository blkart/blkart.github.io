---
layout: post
title: OpenStack监控平台简析
date: 2015-12-01
categories: blog
tags: [openstack,monitor]
description: OpenStack监控平台简析

---

# 前言

OpenStack 发展至今，已经走过了5年多的风风雨雨，随着时间的推移，OpenStack 项目逐渐成熟，各个核心组件逐渐趋于稳定，OpenStack 正在被越来越多的企业部署到生产环境中。**稳定性和可用性**是企业级云平台的一个重要考核指标，稳定性和可用性如何保证？除了需要对云平台有深入了解并在出现故障时能在最短时间内快速定位排除的运维人员外，还需要有一套优秀的**实时监控平台**，能够实时监控云平台状态，在故障出现的第一时间将故障体现出来，便于运维人员在第一时间定位并排除故障，保证云平台稳定运行，保证服务的可用性。


# 概述

最近一段时间对 Mirantis 公司开发的 OpenStack 自动化部署工具 Fuel 中提供的 OpenStack 监控平台进行了研究，通过这些监控插件我们可以部署一套可以实现对 OpenStack 云平台进行实时监控的监控平台。该平台由数据收集、数据存储及数据展示三部分组成，每一部分功能通过不同的开源软件实现。


# 监控平台简析

### 监控平台可以监控什么？不能监控什么？

该监控平台监控的对象为 OpenStack 平台本身，而不是平台上运行的虚拟机及虚拟机内部运行的应用。


#### 可以监控

* 各 OpenStack 节点系统基本状态监控

  * CPU 利用率
  * MEM、SWAP 使用量
  * Process 数量
  * 登陆用户数
  * 磁盘空间使用量
  * 磁盘 IO
  * 网卡 IO


* OpenStack 各 Controller 节点上各组件服务状态监控

  * API 5XX 错误数量
  * API HTTP Response 时间及数量
  * API 可用性（如：nova api）
  * 服务运行状态（如：openstack-nova-scheduler）
  * 资源状态 （如：运行中的 instance 数量）


* 集群状态监控

  * RabbitMQ 集群状态
  * MySQL 集群状态
  * HAProxy 集群状态
  * Ceph 集群状态


* 附加服务监控

  * Apache 服务状态

* Memcached 服务状态


#### 不能监控

* 虚拟机状态监控（如：某台虚拟机是否宕机）
* 虚拟机内部应用监控

### 监控平台架构

![100.jpg](/img/blog_img/100.jpg)

### 监控平台架构分析

整个监控平台由三部分组成（参考上图）：数据收集部分、数据存储部分及数据展示部分。

* 数据收集

数据收集功能通过 Collectd + Heka 两个开源软件实现，Collectd 主要负责系统基本状态信息、OpenStack 各服务状态信息及各集群状态信息的收集，将收集到的信息发送给 Heka；Heka 主要负责 OpenStack 各组件日志的收集，同时对 Collectd 收集到的信息及 OpenStack 各组件日志信息进行解码处理，然后将处理过的数据实时发送到 Influxdb 数据库。Collectd 及 Heka 均支持自定义插件，因此我们可以通过自定义插件来实现对自己开发的功能的监控。


* 数据存储

数据存储功能通过开源数据库 InfluxDB 实现，InfluxDB 是一个开源，分布式，时间序列，事件，可度量和无外部依赖的数据库。所有 OpenStack 节点的监控数据都存储在该数据库中。该数据库支持HTTP API，用户可以通过 HTTP API 对数据库进行读写操作。


* 数据展示

数据展示通过开源软件 Grafana 实现，部署在 Nginx 上，它可以将从 InfluxDB 数据库中获取到的实时监控数据通过图表的形式展示出来。Grafana 拥有迷倒众生的炫酷界面，当我第一次看到它时，便被它那炫酷的界面所折服。Grafana 支持用户自定义图表，当我们需要对自己开发的功能进行监控时，可以非常方便的增加相关监控图表。


### 监控平台搭建

关于监控平台的搭建， Fuel 中已经集成了自动化部署插件（fuel-plugin-influxdb-grafana、fuel-plugins-lma-collector），我们可以使用这些插件实现监控平台的快速自动化部署。


### 监控平台界面

有图有真相，来看下监控平台的几个截图：

![101.jpg](/img/blog_img/101.jpg)

![102.jpg](/img/blog_img/102.jpg)
 

# 总结

综上所述，该监控平台能够为大规模 OpenStack 平台实现实时监控，为云平台的稳定性、可用性提供强有力的保证。目前，监控平台部署插件项目的仓库已经迁移到 OpenStack 仓库，更多的功能逐渐被添加到项目中（如发送报警邮件等），使监控平台越来越强大。

# 相关项目链接

* Collectd：https://github.com/collectd/collectd
* Heka：https://github.com/mozilla-services/heka/
* Influxdb：https://github.com/influxdb/influxdb
* Grafana：https://github.com/grafana/grafana
