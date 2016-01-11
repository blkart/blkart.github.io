---
layout: post
title: Docker Compose 安装
date: 2016-01-11
categories: blog
tags: [Docker,翻译]
description: Docker Compose 安装

---

PS: 继续翻译安装篇，[原文链接](https://docs.docker.com/compose/install/)

## 安装 Docker Compose

你可以在 OS X 和 64-bit Linux 系统上运行 Compose。在安装 Docker Compose 前你需要先安装 Docker。

根据以下步骤安装 Compose：

#### 安装 Docker Engine 1.7.1或以上版本

* [Mac OS X 安装方法](https://docs.docker.com/engine/installation/mac/)

* [Ubuntu 安装方法](https://docs.docker.com/engine/installation/ubuntulinux/)

* [CentOS 安装方法](http://blog.blkart.org/blog/2015/12/12/docker-quick-start/#docker--1)

* [其他操作系统安装方法](https://docs.docker.com/engine/installation/)

#### Mac OS X 系统用户已经安装完成。其他系统用户需要继续完成后面的步骤。

#### 打开 GitHub 上 Compose [发布页面](https://github.com/docker/compose/releases)。

#### 执行以下命令下载 Compose 到本地

```
curl -L https://github.com/docker/compose/releases/download/1.5.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```
#### 为文件增加可执行权限

```
chmod +x /usr/local/bin/docker-compose
```

> 注意：
>
> 需要使用```sudo -i```或以管理员用户身份执行以上命令。

#### 可选步骤，安装为```bash```及```zsh```安装[命令自动补全](https://docs.docker.com/compose/completion/)

#### 测试是否安装成功

    $ docker-compose --version
    docker-compose version: 1.5.2

## 其他安装方式

### 使用 pip 安装

你可以使用 pip 从 pypi 安装 Compose。如果你使用 pip 安装 Compose，强烈建议使用虚拟环境（[virtualenv](https://virtualenv.pypa.io/en/latest/)），因为很多操作系统的 python 系统包与 docker-compose 的依赖关系冲突。虚拟环境（virtualenv）[教程](http://docs.python-guide.org/en/latest/dev/virtualenvs/)。

```
$ pip install docker-compose
```

> 注意：pip 需要 6.0 或以上版本

### 通过容器（container）安装

Compose 也可以运行在一个容器中，通过一个小的 bash 脚本包装。运行以下命令进行安装：

    $ curl -L https://github.com/docker/compose/releases/download/1.5.2/run.sh > /usr/local/bin/docker-compose
    $ chmod +x /usr/local/bin/docker-compose

## Master 版本

如果你有兴趣尝试预发布版本（pre-release）你可以从[这里下载](https://dl.bintray.com/docker-compose/master/)二进制文件。预发布版本可以让你在新版本正式发布前提前尝试新的特性，但也许不是太稳定。

## 升级

如果你从 1.2 或更早的版本进行升级，你需要在 Compose 升级完成后删除或迁移已经存在的容器（containers）。这是因为 1.3 版本中，Compose 使用 Docker 标签（labels）保持容器跟踪（track），所以必须重新创建容器以添加标签。

如果 Compose 检测到没有标签的容器，在你结束它们前 Compose 将拒绝运行。如果你想要继续使用已经存在的容器（例如：它们包含你想保留的数据卷），你可以使用以下命令迁移它们。

```
$ docker-compose migrate-to-labels
```

另外，如果你不需要保留它们，你可以删除它们。Compose 将会创建一个新的。

```
$ docker rm -f -v myapp_web_1 myapp_db_1 ...
```

## 卸载

如果你使用```curl```安装，可以用以下命令卸载：

```
$ rm /usr/local/bin/docker-compose
```

如果你使用```pip```安装，可以用以下命令卸载：

```
$ pip uninstall docker-compose
```


