---
layout: post
title: Docker学习笔记『安装及基本使用方法』
date: 2014-12-14
categories: blog
tags: [Docker]
description: Docker 安装及基本使用方法

---

## 安装

### 基础环境

#### 目前常见的各种操作系统及云平台中都可以安装Docker Engine，如Mac OS X、Linux各种发行版本（Fedora/Ubuntu/Gentoo/Arch等）、Microsof Windows、Google Cloud Platform、Amazon EC2等。本次学习环境选择了Fedora20作为基础环境。

### 安装软件包

```# yum -y install docker-io```

### 启动服务/设置服务随系统启动

```# systemctl start docker
# systemctl enable docker```

## 使用

### 注册帐号/登陆

```
# docker login
Username: ******
Password:
Email: *******@***.com
Account created. Please use the confirmation link we sent to your e-mail to activate it.
```

```
# docker login

Username (*****):
Login Succeeded
```

### 查看可用images

*   初始环境本地无任何image
```
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE 
```

### pull image到本地

*   以centos为例
```
# docker pull centos
centos:latest: The image you are pulling has been verified

511136ea3c5a: Pulling fs layer
511136ea3c5a: Download complete
5b12ef8fd570: Download complete
ae0c2d0bdc10: Download complete
Status: Downloaded newer image for centos:latest 
```

### 查看可用images

```
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
centos              centos7             ae0c2d0bdc10        2 weeks ago         224 MB
centos              latest              ae0c2d0bdc10        2 weeks ago         224 MB 
```

### 在docker中运行程序

*   SELinux为Enforcing时会由于权限问题导致容器启动失败
```
# docker run centos:latest /bin/echo 'Hello world'
permission denied2014/11/23 11:29:05 Error response from daemon: Cannot start container e070fb72d5f29154e02c051d7ea9313c2110547b7702c1cfcb016042081d2934: permission denied 
```

*   此时将SELinux设为permissive状态即可

```
# setenforce 0
```

*   再次运行容器，权限问题解决
```
# sudo docker run centos:latest /bin/echo 'Hello world'

Hello world 
```

### 运行一个交互式的容器

*   -t：在新容器内指定一个伪终端或终端
*   -i：允许我们对容器内的STDIN进行交互

```
# docker run -t -i centos:latest /bin/bash
[root@7377b556e8d5 /]#
[root@7377b556e8d5 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  selinux  srv  sys  tmp  usr  var 
```

### 守护进程式运行容器

*   -d：告诉docker以后台模式运行容器

```
# docker run -d centos:latest /bin/sh -c "while true; do echo hello world; sleep 1; done"
046651b6dd00bf3c497bc47a1c7b365d79672bace262982f0e8a4cd3afd99560 
```

### 查看docker进程

```
# docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
046651b6dd00        centos:centos7      "/bin/sh -c 'while t   2 minutes ago       Up 2 minutes                            ecstatic_sammet      
```

### 停止docker进程

*   此处“ecstatic_sammet”为容器名
```
# docker stop ecstatic_sammet
```

### 查看docker版本

```
# docker version
Client version: 1.3.1
Client API version: 1.15
Go version (client): go1.3.3
Git commit (client): 4e9bbfa/1.3.1
OS/Arch (client): linux/amd64
Server version: 1.3.1
Server API version: 1.15
Go version (server): go1.3.3
Git commit (server): 4e9bbfa/1.3.1 
```

### 运行一个web应用

*   -P：通知Docker所需的网络端口映射从主机映射到容器内
```
# docker run -d -P training/webapp python app.py
Unable to find image 'training/webapp' locally
Pulling repository training/webapp
31fa814ba25a: Download complete
511136ea3c5a: Download complete
f10ebce2c0e1: Download complete
82cdea7ab5b5: Download complete
5dbd9cb5a02f: Download complete
74fe38d11401: Download complete
64523f641a05: Download complete
0e2afc9aad6e: Download complete
e8fc7643ceb1: Download complete
733b0e3dbcee: Download complete
a1feb043c441: Download complete
e12923494f6a: Download complete
a15f98c46748: Download complete
Status: Downloaded newer image for training/webapp:latest
f808ab5984ee4cf5788217b5091bf30bc7ded05ef71dd1827612641a287c4138 
```

### 查看web应用容器

*   -l：返回最后的容器的状态
*   此时可以看到主机的49153号端口映射到容器的5000端口

```
# docker ps -l
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
f808ab5984ee        training/webapp:latest   "python app.py"     6 minutes ago       Up 6 minutes        0.0.0.0:49153-&gt;5000/tcp   cocky_thompson 
```

### 访问主机的49153端口即可访问容器中5000端口对应的服务

```
# curl http://192.168.3.115:49153
Hello world! 
```

### 另外一种查看端口映射的方法

```
# docker port cocky_thompson 5000
0.0.0.0:49153 
```

### 查看web应用的日志

*   -f：使用tail -f来查看容器标准输出
```
# docker logs -f cocky_thompson
* Running on http://0.0.0.0:5000/
192.168.3.115 - - [23/Nov/2014 06:48:51] "GET / HTTP/1.1" 200 -
192.168.3.115 - - [23/Nov/2014 06:48:51] "GET /favicon.ico HTTP/1.1" 404 -
192.168.3.115 - - [23/Nov/2014 06:48:51] "GET /favicon.ico HTTP/1.1" 404 -
192.168.3.115 - - [23/Nov/2014 06:50:02] "GET / HTTP/1.1" 200 - 
```

### 查看容器的进程

```
# docker top cocky_thompson
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                13083               4174                0                   14:39               ?                   00:00:00            python app.py 
```

### 查看docker的底层信息

```
# docker inspect cocky_thompson
    [{
        "Args": [
            "app.py"
        ],
        "Config": {
            "AttachStderr": false,
            "AttachStdin": false,
            "AttachStdout": false,
            "Cmd": [
                "python",
                "app.py"
            ],
            "CpuShares": 0,
            "Cpuset": "",
            "Domainname": "",
            "Entrypoint": null,
            "Env": [
                "HOME=/",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "ExposedPorts": {
                "5000/tcp": {}
            },
            "Hostname": "f808ab5984ee",
            "Image": "training/webapp",
            "Memory": 0,
            "MemorySwap": 0,
            "NetworkDisabled": false,
            "OnBuild": null,
            "OpenStdin": false,
            "PortSpecs": null,
            "SecurityOpt": null,
            "StdinOnce": false,
            "Tty": false,
            "User": "",
            "Volumes": null,
            "WorkingDir": "/opt/webapp"
        },
        "Created": "2014-11-23T06:39:46.951678024Z",
        "Driver": "devicemapper",
        "ExecDriver": "native-0.2",
        "HostConfig": {
            "Binds": null,
            "CapAdd": null,
            "CapDrop": null,
            "ContainerIDFile": "",
            "Devices": [],
            "Dns": null,
            "DnsSearch": null,
            "ExtraHosts": null,
            "Links": null,
            "LxcConf": [],
            "NetworkMode": "bridge",
            "PortBindings": {},
            "Privileged": false,
            "PublishAllPorts": true,
            "RestartPolicy": {
                "MaximumRetryCount": 0,
                "Name": ""
            },
            "VolumesFrom": null
        },
        "HostnamePath": "/var/lib/docker/containers/f808ab5984ee4cf5788217b5091bf30bc7ded05ef71dd1827612641a287c4138/hostname",
        "HostsPath": "/var/lib/docker/containers/f808ab5984ee4cf5788217b5091bf30bc7ded05ef71dd1827612641a287c4138/hosts",
        "Id": "f808ab5984ee4cf5788217b5091bf30bc7ded05ef71dd1827612641a287c4138",
        "Image": "31fa814ba25ae3426f8710df7a48d567d4022527ef2c14964bb8bc45e653417c",
        "MountLabel": "system_u:object_r:svirt_sandbox_file_t:s0:c94,c642",
        "Name": "/cocky_thompson",
        "NetworkSettings": {
            "Bridge": "docker0",
            "Gateway": "172.17.42.1",
            "IPAddress": "172.17.0.15",
            "IPPrefixLen": 16,
            "MacAddress": "02:42:ac:11:00:0f",
            "PortMapping": null,
            "Ports": {
                "5000/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "49153"
                    }
                ]
            }
        },
        "Path": "python",
        "ProcessLabel": "system_u:system_r:svirt_lxc_net_t:s0:c94,c642",
        "ResolvConfPath": "/var/lib/docker/containers/f808ab5984ee4cf5788217b5091bf30bc7ded05ef71dd1827612641a287c4138/resolv.conf",
        "State": {
            "ExitCode": 0,
            "FinishedAt": "0001-01-01T00:00:00Z",
            "Paused": false,
            "Pid": 13083,
            "Restarting": false,
            "Running": true,
            "StartedAt": "2014-11-23T06:39:47.80654401Z"
        },
        "Volumes": {},
        "VolumesRW": {}
    }
    ]
```

### 获取容器的某一个配置信息

*   -f：使用给定的模板格式显示输出信息
```
# docker inspect -f '{{ .NetworkSettings.IPAddress }}' cocky_thompson
172.17.0.15 
```

### 停止容器

*   查看正在运行的容器
```
# docker ps
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
f808ab5984ee        training/webapp:latest   "python app.py"     33 minutes ago      Up 33 minutes       0.0.0.0:49153-&gt;5000/tcp   cocky_thompson 
```

*   停止某个容器
```
# docker stop cocky_thompson
cocky_thompson 
```

*   再次查看正在运行的容器
```
#docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES 
```

### 启动某个容器

```
# docker start cocky_thompson
cocky_thompson 
```

### 删除容器

*   删除前需要先停止容器运行
```
# docker rm cocky_thompson
cocky_thompson 
```

* * *

#### OK，简单了解了基本使用方法，总的来说感觉docker用起来很方便，很简单，而且发现docker命令对[TABLE]的支持很给力，不只可以提示子命令，而且还可以提示命令参数，比如执行docker start [TABLE]，此时会显示出可以启动的容器信息，直接输入某个容器的前几个字符，然后[TABLE]，ok，同样可以自动补全容器名，非常方便。

#### 简单总结下前面用到的命令：

```
# docker --help
Usage: docker [OPTIONS] COMMAND [arg...]

    images    列出镜像
    inspect   获取一个容器的底层信息

        -f：使用给定的模板格式显示输出信息 

    login     注册/登陆到Docker服务器
    logout    退出登陆
    logs      获取容器的日志信息

        -f：使用tail -f来查看容器标准输出 

    ps        列出容器

        -l：返回最后的容器的状态
        -a：显示所有容器的状态（包括停止的容器） 

    pull      从Docker服务器上下载镜像或源
    rm        删除一个或多个容器
    run       在新的容器中运行命令

        -t：在新容器内指定一个伪终端或终端；
        -i：允许我们对容器内的STDIN进行交互
        -d：告诉docker运行容器在后台模式运行
        -P：通知Docker所需的网络端口映射从主机映射到我们的容器内
        -p：指定主机端口与容器内端口的映射关系 

    start     启动一个已经停止的容器
    stop     停止一个正在运行的容器
    top       查找一个容器对应的进程
    version  查看Docker的版本信息 
```

* * *

#### 参考链接：

*   [Docker中文指南](http://www.widuu.com/chinese_docker/)
