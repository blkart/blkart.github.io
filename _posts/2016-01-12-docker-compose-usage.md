---
layout: post
title: Docker Compose 使用
date: 2016-01-12
categories: blog
tags: [Docker,翻译]
description: Docker Compose 使用

---

PS: 继续翻译使用篇，[原文链接](https://docs.docker.com/compose/gettingstarted/)

## 开始

这个页面展示了通过 Compose 构建一个简单的 Python Web 应用程序。这个应用程序使用 Flask 框架并且将数据存放到 Redis 中（示例中会增加存储在 Redis 中的值）。这里使用 Python 进行演示，即使你不熟悉 Python 也应该能够理解对应的概念。

## 前提条件

确定你已经安装好了 Docker Engine 和 Docker Compose。你不需要安装 Python，它会由一个 Docker image 提供。

## 步骤1：基本配置

#### 创建一个项目目录

    $ mkdir composetest
    $ cd composetest

#### 在项目目录中，用你喜欢的文本编辑器创建一个名为```app.py```的文件

    from flask import Flask
    from redis import Redis
    
    
    app = Flask(__name__)
    redis = Redis(host='redis', port=6379)
    
    
    @app.route('/')
    def hello():
        redis.incr('hits')
        return 'Hello World! I have been seen %s times.' % redis.get('hits')
    
    
    if __name__ == "__main__":
        app.run(host="0.0.0.0", debug=True)

#### 在项目目录中创建名为```requirements.txt```的文件，并添加以下内容：

    flask
    redis

> 这是在定义应用程序的依赖

## 步骤2：创建一个 Docker 镜像（image）

在这一步中，你需要构建一个新的 Docker 镜像。这个镜像包含所有的 Python 应用程序所需要的依赖，包括 Python 自己。

#### 在你的项目目录中创建名为```Dockerfile```的文件，并添加以下内容：

    FROM python:2.7
    ADD . /code
    WORKDIR /code
    RUN pip install -r requirements.txt
    CMD python app.py

> 这个文件告诉 Docker 做以下几件事：
>
> * 先构建一个包含了 Python 2.7 的镜像。
>
> * 然后添加当前目录```.```到镜像中的```/code```路径下。
>
> * 切换工作目录到```/code```。
>
> * 安装 Python 的依赖。
>
> * 设置容器（container）的默认命令为```python app.py```

> 想要了解更多关于如何书写 Dockerfiles 的信息，可以查看[Docker 用户指南](https://docs.docker.com/engine/userguide/dockerimages/#building-an-image-from-a-dockerfile)和[Dockerfile 参考](https://docs.docker.com/engine/reference/builder/)。 

#### 构建镜像

    $ docker build -t web .

> 这条命令会根据当前目录包含的信息构建一个名为```web```的镜像。这个命令会自动查找```Dockerfile```、```app.py```和```requirements.txt```文件。

## 步骤3：定义服务

我们可以使用```docker-compose.yml```文件定义服务的配置：

#### 在项目目录中创建一个名为```docker-compose.yml```的文件，并添加以下内容：

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

> 这个 Compose 文件定义了两个服务：```web``` 和 ```redis```。Web 服务：
>
> * 根据当前目录中的 ```Dockerfile``` 进行构建。
>
> * 暴露容器（container）的5000端口到主机的5000端口。
>
> * 挂载主机的项目目录到容器（container）中的 ```/code``` 目录，使你不需要重新构建镜像就能修改代码。
>
> * 链接 web 服务到 Redis 服务。

> ```redis``` 服务使用从 Docker Hub 下载的最新的 Redis 镜像。

## 步骤4：使用 Compose 构建并且运行你的 app

#### 在你的项目目录中，启动你的应用

    $ docker-compose up
    Pulling image redis...
    Building web...
    Starting composetest_redis_1...
    Starting composetest_web_1...
    redis_1 | [8] 02 Jan 18:43:35.576 # Server started, Redis version 2.8.3
    web_1   |  * Running on http://0.0.0.0:5000/
    web_1   |  * Restarting with stat

> Compose 下载一个 Redis 镜像，为你的代码构建一个镜像，并且启动你定义的服务。

#### 在浏览器中输入 ```http://0.0.0.0:5000/``` 查看应用是否正在运行。

> 如果你使用 Linux 主机上的 Docker，web 应用会监听运行 Docker 守护程序的主机的5000端口。如果```http://0.0.0.0:5000```没有被解析，你可以尝试访问```http://localhost:5000```。

> 如果你使用 Mac 上的 Docker Machine，使用 ```docker-machine ip MACHINE_VM``` 来获取你的 Docker 主机的 IP 地址。然后，使用浏览器访问 ```http://MACHINE_VM_IP:5000```。

你将会在浏览器中看到以下信息：

    Hello World! I have been seen 1 times.

刷新页面，数字将会增加。

## 步骤5：尝试下其他命令

如果哦你想在后端运行你的服务，你可以传递 ```-d``` （detached）给 ```docker-compose up``` 命令，可以使用 ```docker-compose ps``` 查看当前正在运行的服务。

    $ docker-compose up -d
    Starting composetest_redis_1...
    Starting composetest_web_1...
    $ docker-compose ps
    Name                 Command            State       Ports
    -------------------------------------------------------------------
    composetest_redis_1   /usr/local/bin/run         Up
    composetest_web_1     /bin/sh -c python app.py   Up      5000->5000/tcp

```docker-compose run``` 命令可以用来在服务中运行一次性命令。例如，查看 ```web``` 服务中可用的环境变量。

    $ docker-compose run web env

通过 ```docker-compose --help``` 可以查看其它可用的命令。你也可以为 bash 和 zsh shell 安装[命令补全](https://docs.docker.com/compose/completion/)，同样也可以用来查看可用的命令。

如果你使用 ```docker-compose up -d``` 命令启动了你的服务，你也许想要在完成实验是停止它们：

    $ docker-compose stop

***

以上就是 Compose 的基本使用方法。
