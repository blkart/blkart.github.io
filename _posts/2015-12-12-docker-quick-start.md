---
layout: post
title: Docker快速入门
date: 2015-12-12
categories: blog
tags: [Docker]
description: Docker快速入门

---

PS: 时间飞逝，距离上次写关于 Docker 的 Blog 已经有一年了，在这一年的时间里，Docker 发展飞速，很多地方由于版本的升级跟以前的使用方法也有了区别，在这里再来一篇最新的 Docker 快速入门指南，希望能帮助目前不了解 Docker 但又想快速了解并掌握 Docker 的基本使用方法的朋友提供些许帮助。

## 概述

Docker是一个为开发者与系统管理员提供的构建、交付及运行应用程序的开放平台。它包括两个部分：Docker Engine——一个轻便的、轻量级运行和封装的工具（用于Images的管理及承载Containers的运行）；Docker Hub——一个用于共享应用和自动化流程的云服务平台（存放了很多其它用户发布的Images）。Docker实现了应用程序的快速部署，并且避免了开发、测试和生产环境之间的冲突。最终得到的：IT人可以在笔记本电脑、数据中心及任何的云之间更快的、保持不变的交付和运行相同的应用程序。

## 存在的原因

### 开发者为什么喜欢它

通过Docker，开发者可以使用任何一种工具链、任何一种语言构建任何一种应用程序。“Docker化”的应用程序是完全可移植的，它可以运行在任何地方——同事的OS X、Windows笔记本、云中运行Ubuntu的测试服务器、数据中心生产环境中运行RHEL的虚拟机。

开发者可以快速启动Docker Hub上的超过13,000个应用程序，Docker可以管理、跟踪修改和依赖，这可以让系统管理员更容易的了解开发者对应用程序的构建工作。通过Docker Hub，开发者可以通过公共或私有的仓库自动化构建流程，同时可以与合作者共享工件。

Docker帮助开发者更快的构建和交付高质量的应用程序。

### 系统管理员为什么喜欢它

系统管理员使用Docker为开发、测试和产品团队提供标准化环境，减少了“在我机器上工作”的指责。通过“Docker化”应用程序平台及它的依赖，系统管理员抽象化了操作系统发行版和底层基础设施之间的差异。

此外，标准化的Docker Engine作为部署单元给予系统管理员在工作负载上的灵活性。不论内部部署裸机或者数据中心的虚拟机或者公有云，工作负载不易受基础架构技术的约束，并改为按业务优先级和策略驱动。此外，Docker Engine的轻量级运行实现了需求变化的向上和向下扩展的快速相应。

Docker可以帮助系统管理员在任何基础设施上快速可靠的运行任何应用程序。

## 与虚拟化的区别

### 虚拟机

{{ :eayunstack:docker:docker-001.png?500 |}}

每一个虚拟机不仅仅包含应用程序（这也许只是MB级）和必要的二进制文件和库，而是整个操作系统（这可能是GB级的）

### Docker

{{ :eayunstack:docker:docker-002.png?500 |}}

Docker Engine容器仅包含应用程序和它的依赖。它在主机操作系统中的用户空间中作为一个独立的进程运行，与其他容器共享内核。因此，它具有与虚拟机一样的资源隔离与分配的优点，而且更便携和高效。

## Docker 系统架构

{{ :eayunstack:docker:docker-003.png?600 |}}

>  **Docker Hub：** 用于发布Docker Image，用户可以从Docker Hub上获取其它用户发布的Image，也可以在上面发布自己的Image；Docker社区提供了免费使用的Docker Hub，用户也可以自己搭建Docker Hub。
>  **Docker Engine：** 用于管理 Docker Image 及承载 Container。
>  **Docker Client：** Docker Engine 的客户端管理命令，它可以跟Docker Engine处在同一个机器上，也可以处在不同的机器上。

## Docker 环境部署

### 环境需求

* 2G MEM + 100G DISK 4Core CPU

* 安装 CentOS7 系统

* 能连接外网

### 安装 Docker Engine

#### 更新系统软件包

    # yum -y update

#### 配置 Docker YUM 源

    # vim /etc/yum.repos.d/docker.repo
    [dockerrepo]
    name=Docker Repository
    baseurl=http://yum.dockerproject.org/repo/main/centos/$releasever/
    enabled=1
    gpgcheck=0

#### 安装 docker-engine 软件包

    # yum install docker-engine

#### 启动 docker 服务

    # systemctl start docker
    # systemctl enable docker

#### 执行命令，确认 docker-engine 已经安装并启动成功

    # docker info
    Containers: 0
    Images: 0
    Server Version: 1.9.1
    Storage Driver: devicemapper
     Pool Name: docker-253:1-201865241-pool
     Pool Blocksize: 65.54 kB
     Base Device Size: 107.4 GB
     Backing Filesystem: xfs
     Data file: /dev/loop0
     Metadata file: /dev/loop1
     Data Space Used: 53.67 MB
     Data Space Total: 107.4 GB
     Data Space Available: 52 GB
     Metadata Space Used: 606.2 kB
     Metadata Space Total: 2.147 GB
     Metadata Space Available: 2.147 GB
     Udev Sync Supported: true
     Deferred Removal Enabled: false
     Deferred Deletion Enabled: false
     Deferred Deleted Device Count: 0
     Data loop file: /var/lib/docker/devicemapper/devicemapper/data
     Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
     Library Version: 1.02.93-RHEL7 (2015-01-28)
    Execution Driver: native-0.2
    Logging Driver: json-file
    Kernel Version: 3.10.0-123.el7.x86_64
    Operating System: CentOS Linux 7 (Core)
    CPUs: 8
    Total Memory: 1.797 GiB
    Name: docker
    ID: GWO6:TQIF:W6JR:WXKC:AGQ7:OM7D:YHSJ:AXY7:HU2E:CGIF:UNBS:MZ2L
    WARNING: bridge-nf-call-iptables is disabled
    WARNING: bridge-nf-call-ip6tables is disabled

>  查看到类似以上示例信息时，确认docker-engine已安装成功

## Docker 的基本使用

### 获取 Image

Image的获取有两种方式：一种是从Docker Hub上下载，另外一种是从TarBall中导入（TarBall内保存的是从其它环境导出的Image）。

#### 从Dcoker Hub上下载Image

* 搜索Image

>  使用该命令默认在 Docker Hub 上搜索指定 Image（需连接外网）

    # docker search centos

* 下载 Image

>  使用该命令默认在 Docker Hub 上下载指定 Image（需连接外网）

    # docker pull centos

#### 从TarBall导入Image

>  此处的TarBall需要提前获得，关于如何制作TarBall可以参看后续关于Image发布的章节

    # docker load -i centos-latest-docker-image.tar

### 查看Images列表

    # docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    centos              latest              ce20c473cd8a        9 weeks ago         172.3 MB

### 创建 Container

#### 创建一个 Container

>  基于名为 centos 的 Image 创建一个新的 Container，同时在 Container 中执行“/bin/bash”命令获取一个shell

    # docker run -ti centos /bin/bash

>  Container 启动成功后命令行会进入到 Container 中，如需返回到宿主机，执行“exit”命令即可。

#### 创建一个持续运行的 Container

>  前面一个例子中的 Container 在执行“exit”退出后，Container 即停止运行（stop），原因为 Container 会随着其内部所运行的进程的停止而停止。下面的示例为启动一个持续运行的 Container。

    # JOB=$(docker run -d centos /bin/sh -c "while true;do echo hello world; sleep 1; done")

>  **“JOB“变量内保存的是 Container 的 ID，使用 docker 命令对该 Container 进行管理时需要用到该 ID**

>  下面的命令可以查看 Container 中运行的进程的输出信息

    # docker logs $JOB

>  使用以下命令可将运行中的 Container 停止

    # docker kill $JOB

### 查看 Container 列表

    # docker ps # only list running containers
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    936db2e8b020        centos              "/bin/sh -c 'while tr"   5 days ago          Up 4 seconds                                 backstabbing_hugle

    # docker ps -a # list all containers
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS                    NAMES
    c9b280036c7d        centos              "/bin/bash"              5 days ago          Exited (0) 5 days ago                              kickass_mahavira
    936db2e8b020        centos              "/bin/sh -c 'while tr"   5 days ago          Up About a minute                                  backstabbing_hugle

### Container 的基本管理

#### 启动停止状态的 Container

    # docker start $JOB

#### 停止运行状态的 Container

    # docker stop $JOB

#### 重启 Container

    # docker restart $JOB

#### 删除 Container

    # docker rm $JOB

## 制作 Docker Image

Docker Image 的制作有两种方式：docker commit 和 docker build。docker commit 指通过手动启动一个Container，在容器中部署对应的应用，通过docker commit命令将该Container封装成一个Image；docker build 指通过创建一个应答文件，在应答文件中预定义Image制作过程中所执行的动作，通过docker build命令根据应答文件自动化封装Image（其实就是将docker commit所执行的操作自动化）。

### docker commit

#### 启动新的Container

    # docker run -ti centos /bin/bash

#### 在Container中部署应用（如安装软件包）

    # yum -y install nmap-ncat
    # exit

#### Commit Container，将该Container封装成Image

    # docker commit <Container ID> <Image Name>

#### 查看Images列表

    # docker images

### docker build

#### 创建工作目录

    # mkdir /docker_image_build
    # cd /docker_image_build

#### 创建应答文件，命名为Dockerfile

>  文件名可以自定义，如果自定义文件名，执行"docker build"命令时需使用"-f <file name>"指定文件名。

    # vim Dockerfile
    FROM centos
    MAINTAINER blkart <blkart.org@gmail.com>
    RUN yum -y install httpd
    ADD start.sh /usr/local/bin/start.sh
    ADD index.html /var/www/html/index.html

>  "FROM" : 定义基于本地名为 "centos" 的 Tag 为 "latest" 的 Image 创建 Container
>  "MAINTAINER" : 标记新Image的维护者信息
>  "RUN" : 在Container中执行的命令
>  "ADD" : 将工作目录中的文件添加到Container中的指定位置

>  关于Dockerfile的书写方法可参考：https://docs.docker.com/engine/articles/dockerfile_best-practices/

#### 创建 start.sh 脚本

>  该脚本用于在Container启动时启动httpd服务

    # echo "/usr/sbin/httpd -DFOREGROUND" > start.sh
    # chmod a+x start.sh

#### 创建 index.html 文件

    # echo "docker image build test" > index.html

#### Build Image

    # docker build -t blkart/centos:httpd .
    Sending build context to Docker daemon 4.096 kB
    Step 1 : FROM centos
     ---> ce20c473cd8a
    Step 2 : MAINTAINER blkart <blkart.org@gmail.com>
     ---> Running in d9c855a7ca16
     ---> a8223a6b8a8f
    Removing intermediate container d9c855a7ca16
    Step 3 : RUN yum -y install httpd
     ---> Running in 9fcdfde6f85b
    Loaded plugins: fastestmirror
    Determining fastest mirrors
     * base: mirrors.btte.net
     * extras: mirrors.yun-idc.com
     * updates: mirrors.yun-idc.com
    Resolving Dependencies
    --> Running transaction check
    ---> Package httpd.x86_64 0:2.4.6-31.el7.centos.1 will be installed
    --> Processing Dependency: httpd-tools = 2.4.6-31.el7.centos.1 for package: httpd-2.4.6-31.el7.centos.1.x86_64
    --> Processing Dependency: system-logos >= 7.92.1-1 for package: httpd-2.4.6-31.el7.centos.1.x86_64
    --> Processing Dependency: /etc/mime.types for package: httpd-2.4.6-31.el7.centos.1.x86_64
    --> Processing Dependency: libaprutil-1.so.0()(64bit) for package: httpd-2.4.6-31.el7.centos.1.x86_64
    --> Processing Dependency: libapr-1.so.0()(64bit) for package: httpd-2.4.6-31.el7.centos.1.x86_64
    --> Running transaction check
    ---> Package apr.x86_64 0:1.4.8-3.el7 will be installed
    ---> Package apr-util.x86_64 0:1.5.2-6.el7 will be installed
    ---> Package centos-logos.noarch 0:70.0.6-3.el7.centos will be installed
    ---> Package httpd-tools.x86_64 0:2.4.6-31.el7.centos.1 will be installed
    ---> Package mailcap.noarch 0:2.1.41-2.el7 will be installed
    --> Finished Dependency Resolution
    Dependencies Resolved
    ================================================================================
     Package           Arch        Version                       Repository    Size
    ================================================================================
    Installing:
     httpd             x86_64      2.4.6-31.el7.centos.1         updates      2.7 M
    Installing for dependencies:
     apr               x86_64      1.4.8-3.el7                   base         103 k
     apr-util          x86_64      1.5.2-6.el7                   base          92 k
     centos-logos      noarch      70.0.6-3.el7.centos           updates       21 M
     httpd-tools       x86_64      2.4.6-31.el7.centos.1         updates       79 k
     mailcap           noarch      2.1.41-2.el7                  base          31 k
    Transaction Summary
    ================================================================================
    Install  1 Package (+5 Dependent packages)
    Total download size: 24 M
    Installed size: 31 M
    Downloading packages:
    warning: /var/cache/yum/x86_64/7/base/packages/apr-1.4.8-3.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
    Public key for apr-1.4.8-3.el7.x86_64.rpm is not installed
    Public key for httpd-tools-2.4.6-31.el7.centos.1.x86_64.rpm is not installed
    --------------------------------------------------------------------------------
    Total                                              1.2 MB/s |  24 MB  00:20     
    Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    Importing GPG key 0xF4A80EB5:
     Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
     Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
     Package    : centos-release-7-1.1503.el7.centos.2.8.x86_64 (@CentOS)
     From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    Running transaction check
    Running transaction test
    Transaction test succeeded
    Running transaction
      Installing : apr-1.4.8-3.el7.x86_64                                       1/6 
      Installing : apr-util-1.5.2-6.el7.x86_64                                  2/6 
      Installing : httpd-tools-2.4.6-31.el7.centos.1.x86_64                     3/6 
      Installing : centos-logos-70.0.6-3.el7.centos.noarch                      4/6 
      Installing : mailcap-2.1.41-2.el7.noarch                                  5/6 
      Installing : httpd-2.4.6-31.el7.centos.1.x86_64                           6/6 
      Verifying  : apr-1.4.8-3.el7.x86_64                                       1/6 
      Verifying  : mailcap-2.1.41-2.el7.noarch                                  2/6 
      Verifying  : apr-util-1.5.2-6.el7.x86_64                                  3/6 
      Verifying  : httpd-tools-2.4.6-31.el7.centos.1.x86_64                     4/6 
      Verifying  : centos-logos-70.0.6-3.el7.centos.noarch                      5/6 
      Verifying  : httpd-2.4.6-31.el7.centos.1.x86_64                           6/6 
    Installed:
      httpd.x86_64 0:2.4.6-31.el7.centos.1                                          
    Dependency Installed:
      apr.x86_64 0:1.4.8-3.el7                                                      
      apr-util.x86_64 0:1.5.2-6.el7                                                 
      centos-logos.noarch 0:70.0.6-3.el7.centos                                     
      httpd-tools.x86_64 0:2.4.6-31.el7.centos.1                                    
      mailcap.noarch 0:2.1.41-2.el7                                                 
    Complete!
     ---> 40dda8cc02a2
    Removing intermediate container 9fcdfde6f85b
    Step 4 : ADD start.sh /usr/local/bin/start.sh
     ---> 78a4b58c0be8
    Removing intermediate container 22351e458f20
    Step 5 : ADD index.html /var/www/html/index.html
     ---> 1b2e2d52c06d
    Removing intermediate container be35e8dfcf7a
    Successfully built 1b2e2d52c06d

#### 查看 Images 列表

    # docker images

## Docker Image 的发布

Docker Image 制作完成后，我们如何方便的将Image提供给其它人使用？这时候就需要将Image发布出去，在这里介绍两种发布方式：通过简单的WEB服务发布；通过Docker Hub发布。

### 通过WEB服务发布Image

#### 首先，我们要搭建一个WEB服务器，并在WEB服务器上创建用于存放Images的目录，关于WEB服务器的搭建，这里不再赘述。

#### 将要发布的Image导出到TarBall中

    # docker save -o centos-httpd-docker-image.tar centos:httpd

#### 将TarBall存放到WEB服务器上

    # scp centos-httpd-docker-image.tar xx.xx.xx.xx:/var/www/html/docker-images

### 通过Docker Hub发布Image

Docker Hub是一个用于存放Docker Image的仓库，Docker社区在公网中为我们免费提供了Docker Hub，我们可以将自己制作的Image发布到Docker社区提供的Docker Hub上去。

>  如果有需求，我们也可以自己搭建Docker Hub。本示例使用Docker社区提供的Docker Hub。

#### 登陆Docker社区提供的Docker Hub，注册帐号，并创建Image仓库。

https://hub.docker.com/

#### 在命令行连接Docker Hub

    # docker login -u userabc -p abc123 -e userabc@gmail.com

#### 将Image Push到Docker Hub

    # docker push centos:httpd

#### 登陆到Docker Hub确认Image发布成功

## Docker 的高级使用

### Container 端口映射

当我们在 Container 中运行了一个应用时，这个应用也许会通过某个端口向外提供服务，用户如果在网络中的其它机器上访问到 Container 中提供的服务呢？我们在创建 Container 时需要将宿主机的某个端口跟 Container 中的某个端口进行映射，映射后，用户访问宿主机的某个端口，即可访问到该端口映射到的某个 Container 中的某个端口。

#### 创建 Container

>  以下示例中创建一个封装了httpd服务的 Container，并将宿主机的tcp 9000端口映射到该 Container 的tcp 80端口。其它用户访问宿主机9000端口时，访问到的是 Container 的80端口。

    # docker run -d -p 9000:80 centos:httpd /bin/sh -c /usr/local/bin/start.sh

#### 访问测试

>  此处__192.168.5.21__为宿主机的IP地址

    # curl http://192.168.5.21:9000
    docker image build test

#### 访问正在运行的 Container

以上示例中的 Container 创建成功后会在后台运行，当我们需要对 Container 中的某些文件进行修改时，就需要访问正在运行的 Container。

    [root@docker-engine ~]# docker exec -ti <Container ID | NAME> /bin/bash
    [root@936db2e8b020 /]#

#### 在 Container 中创建文件

    [root@936db2e8b020 /]# echo XXX > /var/www/html/test.html
    [root@936db2e8b020 /]# exit

#### 访问测试

    # curl http://192.168.5.21:9000/test.html
    XXX
