---
layout: post
title: Fuel6部署OpenStack juno（on CentOS6.5）测试笔记『2』
date: 2015-01-23
categories: blog
tags: [openstack,juno]
description: Fuel6部署OpenStack juno（on CentOS6.5）测试笔记

---

**前一篇里介绍了使用Fuel自动化部署OpenStack环境，环境已经部署完成，这节来对部好的环境进行一些简单的探索。**

### 目录

* 命令行下访问节点
* 各节点网络配置总结
* WEB访问OpenStack Dashboard

## 命令行下访问节点

* 首先，回忆下上一篇中的网络拓扑，Fuel节点通过“PXE”网络跟OpenStack环境中的所有节点进行连接，“PXE”网络又名“Admin（PXE）”网络，从名字就可以看出，“PXE”网络也是Fuel主机对OpenStack环境的管理网络，因此可以在Fuel主机通过该网络访问各个主机。
* 其次，PXE网络DHCP地址池从10.10.0.3开始，因此第一个节点PXE网卡的IP为10.10.0.3，其余节点依次类推。
* 最后，在我们部署OpenStack环境前的配置中（Fuel WEB界面“设置”面板），配置了“Public Key”，在OpenStack环境部署时，该“key”已被存放到所有节点的/root/.ssh/authorized_keys文件中，因此Fuel主机可以通过ssh密钥对验证访问所有节点。

综上所述，我们可以在Fuel节点访问各个节点，如访问第一个节点：

    # ssh 10.10.0.3
    Warning: Permanently added '10.10.0.3' (RSA) to the list of known hosts.
    Last login: Fri Jan 23 08:47:31 2015 from 10.10.0.2

> 可使用相同方法访问其它节点。

连接到controller节点后，我们可以通过命令行对OpenStack环境进行管理

#### 首先配置环境变量脚本

    # vim admin-openrc.sh
    export OS_TENANT_NAME=admin
    export OS_USERNAME=admin
    export OS_PASSWORD=admin
    export OS_AUTH_URL=http://10.0.0.2:35357/v2.0

#### 加载环境变量

    # source admin-openrc.sh

#### 列出images

    # nova image-list

#### 可使用其它OpenStack管理命令对OpenStack环境进行管理

## 各节点网络配置总结

连接到各个节点后，可以使用ifconfig或ip命令来查看网络配置信息，本次环境配置信息如下：

<table>
  <thead>
    <tr>
      <th>
        角色
      </th>

      <th>
        节点
      </th>

      <th>
        网卡
      </th>

      <th>
        IP地址
      </th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>
        controller
      </td>

      <td>
        node-1
      </td>

      <td>
        br-ex
br-mgmt
br-storage
br-fw-admin
**br-ex-hapr**
**br-mgmt-hapr**
      </td>

      <td>
        10.0.0.3/24
10.10.1.3/24
10.10.3.2/24
10.10.0.3/24
**10.0.0.3/24**
**10.10.1.3/24**
      </td>
    </tr>

    <tr>
      <td>
        ----
      </td>

      <td>
        node-2
      </td>

      <td>
        br-ex
br-mgmt
br-storage
br-fw-admin
      </td>

      <td>
        10.0.0.4/24
10.10.1.4/24
10.10.3.3/24
10.10.0.4/24
      </td>
    </tr>

    <tr>
      <td>
        ----
      </td>

      <td>
        node-3
      </td>

      <td>
        br-ex
br-mgmt
br-storage
br-fw-admin
      </td>

      <td>
        10.0.0.5/24
10.10.1.5/24
10.10.3.4/24
10.10.0.5/24
      </td>
    </tr>

    <tr>
      <td>
        compute
      </td>

      <td>
        node-4
      </td>

      <td>
        br-mgmt
br-storage
br-fw-admin
      </td>

      <td>
        10.10.1.6/24
10.10.3.5/24
10.10.0.6/24
      </td>
    </tr>

    <tr>
      <td>
        ----
      </td>

      <td>
        node-5
      </td>

      <td>
        br-mgmt
br-storage
br-fw-admin
      </td>

      <td>
        10.10.1.7/24
10.10.3.6/24
10.10.0.7/24
      </td>
    </tr>

    <tr>
      <td>
        ceph
      </td>

      <td>
        node-6
      </td>

      <td>
        br-mgmt
br-storage
br-fw-admin
      </td>

      <td>
        10.10.1.8/24
10.10.3.7/24
10.10.0.8/24
      </td>
    </tr>

    <tr>
      <td>
        ----
      </td>

      <td>
        node-7
      </td>

      <td>
        br-mgmt
br-storage
br-fw-admin
      </td>

      <td>
        10.10.1.9/24
10.10.3.8/24
10.10.0.9/24
      </td>
    </tr>

    <tr>
      <td>
        ----
      </td>

      <td>
        node-8
      </td>

      <td>
        br-mgmt
br-storage
br-fw-admin
      </td>

      <td>
        10.10.1.10/24
10.10.3.9/24
10.10.0.10/24
      </td>
    </tr>
  </tbody>
</table>

## WEB访问OpenStack Dashboard

* 访问http://10.0.0.2

  ![选区_068](/img/blog_img/068.png)

* 使用admin/admin登陆

  ![选区_069](/img/blog_img/069.png)

* 查看images

  ![选区_070](/img/blog_img/070.png)

* 启动一个新的实例

  ![选区_071](/img/blog_img/071.png)

  ![选区_072](/img/blog_img/072.png)

  ![选区_073](/img/blog_img/073.png)

  ![选区_074](/img/blog_img/074.png)
