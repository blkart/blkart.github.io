---
layout: post
title: Docker Compose 用例：Compose 和 Django
date: 2016-01-12
categories: blog
tags: [Docker,翻译]
description: Docker Compose 用例

---

PS: 来几个用例，[原文链接](https://docs.docker.com/compose/django/)

这篇快速入门指南演示了如何使用 Compose 配置和运行一个简单的 Django/PostgreSQL 应用。你需要先安装好 Docker Compose。

## 定义项目组件

对于这个项目，你需要创建一个 Dockerfile，一个 Python 依赖文件，以及一个 ```docker-compose.yml``` 文件。

#### 创建一个空的项目目录

你可以使用一个好记的目录名。这个目录就是你的应用镜像的内容。这个目录只会包含构建这个镜像的资源。

#### 在项目目录中创建一个新的文件 ```Dockerfile```

这个 Dockerfile 通过一个或多个配置镜像的构建命令定义了你的应用程序镜像的内容。构建完成，你可以在一个容器（container）中运行这个镜像。更多关于 ```Dockerfile``` 的信息，可以查看[Docker 用户指南](https://docs.docker.com/engine/userguide/dockerimages/#building-an-image-from-a-dockerfile)和[Dockerfile 参考](https://docs.docker.com/engine/reference/builder/)。

#### 添加以下内容到 ```Dockerfile``` 中

    FROM python:2.7
    ENV PYTHONUNBUFFERED 1
    RUN mkdir /code
    WORKDIR /code
    ADD requirements.txt /code/
    RUN pip install -r requirements.txt
    ADD . /code/

> ```Dockerfile``` 先下载一个 Python 2.7 的基本镜像。然后在这个基本镜像中添加一个新的目录 ```/code```。最后根据 ```requirements.txt``` 文件安装 Python 安装依赖。

#### 保存文件并退出编辑器

#### 在项目目录中创建一个 ```requirements.txt``` 文件

这个文件会被 ```Dockerfile``` 中的 ```RUN pip install -r requirements.txt``` 命令调用。

#### 在 ```requirements.txt``` 文件中添加以下内容

    Django
    psycopg2

#### 保存并退出编辑器

#### 在项目目录中创建 ```docker-compose.yml``` 文件

```docker-compose.yml``` 文件描述了创建应用的服务。这个例子中指的是 web 和 数据库服务。Compose 文件同时也描述了这些服务使用哪个 Docker 镜像、它们之间如何链接以及需要将哪些卷挂载到容器（container）中。最后， ```docker-compose.yml``` 文件描述了这些服务暴露了哪些端口。更多关于这个文件如何工作的信息，可以查看 ```docker-compose.yml``` [参考](https://docs.docker.com/compose/compose-file/)。

#### 先加下面的配置到 ```docker-compose.yml``` 中

    db:
      image: postgres
    web:
      build: .
      command: python manage.py runserver 0.0.0.0:8000
      volumes:
        - .:/code
      ports:
        - "8000:8000"
      links:
        - db

> 这个文件描述了两个服务：```db``` 服务和 ```web``` 服务。

#### 保存并退出编辑器

## 创建一个 Django 项目

在这一步中，你将根据前面过程中定义的构建内容构建镜像来创建 Django 项目。

#### 切换到项目的根目录

#### 使用 ```docker-compose``` 命令创建 Django 项目

    $ docker-compose run web django-admin.py startproject composeexample .

> 这条命令告诉 Compose 使用 web 服务的镜像及配置，并且在容器（container）中运行 ```django-admin.py startproject composeexample``` 命令。因为 ```web``` 镜像目前还不存在，Compose 会根据 ```docker-compose.yml``` 文件中的配置 ```build: .``` 先构建这个镜像。

> 当 ```web``` 服务镜像构建完成后，Compose 会基于它创建一个容器（container），并且在容器中运行 ```djanto-admin.py startproject composeexample``` 命令。这个命令告诉 Django 创建 Django 项目相关的一系列文件和目录。

#### ```docker-compose``` 命令执行完成后，列出项目目录下的内容

    $ ls -l
    drwxr-xr-x 2 root   root   composeexample
    -rw-rw-r-- 1 user   user   docker-compose.yml
    -rw-rw-r-- 1 user   user   Dockerfile
    -rwxr-xr-x 1 root   root   manage.py
    -rw-rw-r-- 1 user   user   requirements.txt

> ```django-admin``` 创建的文件的属于 root 用户。这是因为容器以 root 用户身份运行。

#### 修改新文件的属主

    sudo chown -R $USER:$USER .

## 连接数据库

在这一章节，你会配置 Django 的数据库连接。

#### 编辑项目目录中的 ```composeexample/settings.py``` 文件

#### 使用以下内容替换文件中的 ```DATABASES = ...``` 部分

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': 'postgres',
            'USER': 'postgres',
            'HOST': 'db',
            'PORT': 5432,
        }
    }

> 这个配置通过 ```docker-compose.yml``` 文件中指定的 postgres Docker 镜像来确认。

#### 保存并退出编辑器

#### 运行 ```docker-compose up``` 命令

    $ docker-compose up
    Starting composepractice_db_1...
    Starting composepractice_web_1...
    Attaching to composepractice_db_1, composepractice_web_1
    ...
    db_1  | PostgreSQL init process complete; ready for start up.
    ...
    db_1  | LOG:  database system is ready to accept connections
    db_1  | LOG:  autovacuum launcher started
    ..
    web_1 | Django version 1.8.4, using settings 'composeexample.settings'
    web_1 | Starting development server at http://0.0.0.0:8000/
    web_1 | Quit the server with CONTROL-C.

***

到这里，你的 Django 应用将会监听你的 Docker 主机上的 ```8000``` 端口。如果你使用 Docker Machine VM，你可以使用 ```docker-machine ip MACHINE_NAME``` 来获取 IP 地址。
