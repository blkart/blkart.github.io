---
layout: post
title: Docker Compose 简介
date: 2016-01-11
categories: blog
tags: [Docker]
description: Docker Compose 简介

---

PS: 翻译篇官网简介，[原文链接](https://docs.docker.com/compose/)

## 简介

Compose 是一个用来定义和运行包含多个容器（multi-comtainer）的 Docker 应用程序的工具。你可以使用一个 Compose 文件配置你的服务。然后使用一条命令，就可以根据你配置的配置文件创建和启动所有的服务。想要了解关于 Compose 的更多特性，可以查看 Compose 的[特性列表](https://docs.docker.com/compose/#features)。

## 使用 Compose 的三个步骤

* 使用 Dockerfile 定义你的应用，使你的应用可以在任何地方重复部署。

* 使用 docker-compose.yml 定义组成你的应用的各个服务，使这些服务可以同时运行在一个隔离的环境中。

* 最后，运行 ```docker-compose up``` 命令，Compose 将会启动并且运行你的应用。

## docker-compose.yml 文件示例

    web:
      build: .
      ports:
       - "5000:5000"
      volumes:
       - .:/code
      links:
       - redis
    redis:
      image: redis

> 想了解更多关于 Compose 文件的信息，可以查看[Compose 文件参考](https://docs.docker.com/compose/compose-file/)

## Compose 管理命令

Compose 提供了一些命令用来管理你的应用的整个生命周期。

* 启动、停止及重新构建服务

* 显示运行中的服务的状态

* 输出运行中的服务的日志

* 在一个服务中运行一个一次性的命令

## Compose 特性

#### 在一台主机上构建多个隔离的环境

Compose 使用一个项目名来实现项目之间的隔离，项目名可以用来：

  * 在一台开发主机上，创建一个环境的多份拷贝（例如：针对一个项目的每一个特性分支运行一个稳定拷贝）

  * 在一台 CI 服务器上，为防止构建间的相互干扰，你可以将项目名设置为一个唯一的版本号

  * 在一台共享的开发主机上，为防止不同项目使用相同服务名而发生相互干扰

> 默认配置中，项目以项目目录名命名。你可以使用命令的```-p```选项（或```COMPOSE_PROJECT_NAME```环境变量）自定义项目名。

#### 在容器创建时持久化卷的数据

Compose 持久化你的服务的所有的卷。当```docker-compose up```命令运行时，如果发现之前运行的任何容器，将会从旧的容器中拷贝卷到新的容器中。这样可以保证你之前在卷中创建的所有数据不会丢失。

#### 只在有修改时重新创建容器

Compose 缓存已经创建的容器的配置。当你重启一个没有修改的服务，Compose 会重复使用已经存在的容器。重复使用容器意味着你可以非常快速的修改你的环境。

#### 在环境间变量化和移动一个组合

Compose 支持在 Compose 文件中定义变量。你可以使用这些变量根据不同的环境或用户自定义的你的组合。关于变量[更多信息](https://docs.docker.com/compose/compose-file/#variable-substitution)。

你可以通过```extends```字段或创建多个 Compose 文件来扩展一个 Compose 文件。关于extends[更多信息](https://docs.docker.com/compose/extends/)。

## 通用用例

Compose 有很多不同的用法。下面列举一些通用的用例。

#### 开发环境

当你开发一个软件时，在一个隔离的环境中运行一个应用程序并与之交互的能力是至关重要的。Compose 命令行工具可以用于创建这个环境并与之交互。

Compose 文件提供一种方法来文档化及配置应用程序的服务的依赖（例如：数据库、队列、缓存、web 服务 API 等）。你可以使用 Compose 命令行工具的一条命令（```docker-compose up```）为每一个依赖创建一个或多个容器。

同时，这些特性为开发者提供了开始一个项目的便捷方式。Compose 可以将一个多页的“开发者开始指南”缩小到一个机器可读的 Compose 文件及几个命令中。

#### 自动化测试环境

自动化测试套件是任何一个持续部署或持续集成过程的重要组成部分。自动化端到端（end-to-end）测试需要一个用于运行测试的环境。Compose 为你的测试套件提供了一种便捷的创建和销毁隔离的测试环境的方式。通过在 Compose 文件中完整的定义环境，你可以使用几个命令完成这些环境的创建和销毁。

    $ docker-compose up -d
    $ ./run_tests
    $ docker-compose stop
    $ docker-compose rm -f

#### 单一主机部署

Compose 一直专注于开发和测试的工作流程，但在每个发行的版本中，我们在持续开发更多的面向生产（production-oriented）的特性。你可以使用 Compose 部署一个远端的 Docker Engine。这里的 Docker Engine 也许是 Docker Machine 提供的一个单一实例（instance），或者是整个 Docker Swarm 集群。

> 关于面向生产（production-oriented）的特性，可以[查看这里](https://docs.docker.com/compose/production/)。

## 发行注记

如需查看 Dcoker Compose 过去和当前的发行版本的变更详情，可以参考[CHANGELOG](https://github.com/docker/compose/blob/master/CHANGELOG.md)。

## 获取帮助

Docker Compose 还在活跃的开发。如果你需要帮助、贡献力量或只是想与志同道合的人讨论这个项目。我们社区有一些开放的通道。

* 报告 BUGs 或功能需求：使用 Github 上的 [issue tracker](https://github.com/docker/compose/issues)。

* 与其他人实时讨论项目：可以加入 IRC freenode 上的```#docker-compose```频道。

* 贡献代码或文档修改：可以在 Github 上提交一个 [pull request](https://github.com/docker/compose/pulls)。
