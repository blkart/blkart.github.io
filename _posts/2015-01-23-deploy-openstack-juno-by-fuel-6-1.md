---
layout: post
title: Fuel6部署OpenStack juno（on CentOS6.5）测试笔记『1』
date: 2015-01-23
categories: blog
tags: [openstack]
description: Fuel6部署OpenStack juno（on CentOS6.5）测试笔记

---

**最近尝试用Fuel部署了OpenStack，记录如下:**

### **目录**

*   **部署准备**

        *   **下载Fuel 6安装镜像**
    *   **准备服务器**
    *   **搭建物理网络**
*   **部署流程**

        *   **在Fuel主机上安装操作系统**
    *   **对Fuel主机进行初始化配置**
    *   **访问Fuel的WEB管理界面，创建一个新的OpenStack环境**

# 部署准备

*   下载Fuel 6安装镜像，可到[[这里]](https://software.mirantis.com/)下载，如下图所示：

![选区_015](/img/blog_img/015.png)

![选区_016](/img/blog_img/016.png)

*   准备服务器9台，服务器角色划分及最低配置要求如下：

<table>
<thead>
<tr>
  <th>服务器角色</th>
  <th>CPU</th>
  <th>MEM</th>
  <th>DISK</th>
  <th>NIC</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Fuel</td>
  <td>4</td>
  <td>4G</td>
  <td>100G</td>
  <td>2</td>
</tr>
<tr>
  <td>Controller1</td>
  <td>4</td>
  <td>4G</td>
  <td>100G</td>
  <td>4</td>
</tr>
<tr>
  <td>Controller2</td>
  <td>4</td>
  <td>4G</td>
  <td>100G</td>
  <td>4</td>
</tr>
<tr>
  <td>Controller3</td>
  <td>4</td>
  <td>4G</td>
  <td>100G</td>
  <td>4</td>
</tr>
<tr>
  <td>Compute1</td>
  <td>4</td>
  <td>4G</td>
  <td>100G</td>
  <td>4</td>
</tr>
<tr>
  <td>Compute2</td>
  <td>4</td>
  <td>4G</td>
  <td>100G</td>
  <td>4</td>
</tr>
<tr>
  <td>Ceph1</td>
  <td>4</td>
  <td>2G</td>
  <td>100G/400G</td>
  <td>4</td>
</tr>
<tr>
  <td>Ceph2</td>
  <td>4</td>
  <td>2G</td>
  <td>100G/400G</td>
  <td>4</td>
</tr>
<tr>
  <td>Ceph3</td>
  <td>4</td>
  <td>2G</td>
  <td>100G/400G</td>
  <td>4</td>
</tr>
</tbody>
</table>

    > **提示**
> 
>         *   如果没有9台物理服务器，也可在一台物理服务器上通过虚拟化的方式按以上配置创建9台虚拟机完成该测试。*   本次测试即在一台物理服务器上创建了9台虚拟机完成的（虚拟机需配置复制主机CPU）。 物理服务器的OS为CentOS 7，需要配置KVM嵌套虚拟化（配置方法见**『注 1』**）。
>     *   除Fuel外，其他机器第一引导项均设置为**网络引导**。
*   搭建物理网络，网络拓扑如下，根据拓扑所示使用交换机将各服务器连接到对应的网络中：

![选区_013](/img/blog_img/013.png)

> **注意**
> 
>         *   各服务器网卡连接顺序如图中左上角所示，网卡1、2、3、4分别连接网络PXE、Management、Storage、External网络。

# 部署流程

## 在Fuel主机上安装操作系统

*   在Fuel服务器上使用Fuel 6安装镜像安装操作系统，将ISO镜像放到虚拟机的虚拟光驱中，启动虚拟机，可看到如下图所示内容：

![选区_017](/img/blog_img/017.png)

*   此时按“Tab”键可编辑启动参数，如下图所示：

![选区_018](/img/blog_img/018.png)

*   将最后“**showmenu=no**”修改为“**showmenu=yes**”，如下图所示：

![选区_019](/img/blog_img/019.png)

## 对Fuel主机进行初始化配置

*   系统安装完成后，可看到以下配置界面：

![选区_020](/img/blog_img/020.png)

    *   【Network Setup】修改IP地址为“10.10.0.2”，修改网关为“10.10.0.1”，并选择“Apply”，按回车应用新配置。

![选区_021](/img/blog_img/021.png)

        *   【PXE Setup】如无特殊需求，使用默认配置即可。

        *   【DNS&amp;Hostname】可设置主机名/DNS，根据需求进行配置，如下图所示：

![选区_022](/img/blog_img/022.png)

        *   【Time Sync】NTP服务相关配置，默认配置中指定了三个公网中的NTP服务器，如果Fuel主机可以连接到公网中的NTP服务器，使用默认配置即可；如无法连接到公网中的NTP服务器，需要将服务器指向“127.127.1.0”，如下图所示：

![选区_023](/img/blog_img/023.png)

    > **注意**
> 
>             *   OpenStack环境中各个节点间时间必须是同步的，在Fuel部署的OpenStack环境中所有节点都将Fuel节点设置为NTP服务器进行时间同步，因此Fuel上的NTP服务必须配置正确！*   NTP服务必须与上级时间源进行时间同步后才能向客户端提供NTP服务！因此此处指定的NTP服务器必须可以访问！*   当NTP服务器将上游时间服务器指向“127.127.1.0”时，表示使用本地时钟作为上级时间源。
    *   【Root Password】设置Fuel系统的root用户密码，默认root密码为“r00tme”，可根据需求在此处修改root用户密码。

        *   【Fuel User】此处可设置Fuel WEB管理门户admin用户的密码，默认admin用户密码为“admin”，可根据需求在此处修改admin用户密码。

        *   【Shell Login】可获得一个Shell

        *   【Quit Setup】保存配置并退出，如图所示：

![选区_024](/img/blog_img/024.png)

    > **提示**
> 
>             *   Fuel初始化配置完成后，系统会根据用户所配置的参数对Fuel环境进行初始化，该过程是自动完成的，不需要用户的交互操作。

## 访问Fuel的WEB管理界面，创建一个新的OpenStack环境

*   访问10.10.0.2，使用admin/admin登陆fuel WEB管理界面

![选区_028](/img/blog_img/028.png)

![选区_029](/img/blog_img/029.png)

*   点击“新建OpenStack环境”，创建一个新的OpenStack环境

    *   设置新环境的名称及新环境中要安装的OpenStack的版本及底层操作系统的发行版

![选区_031](/img/blog_img/031.png)

        *   设置环境部署模式，本次部署的环境中需要实现Controller节点的HA，因此选择“HA多节点”

![选区_032](/img/blog_img/032.png)

        *   选择底层的hypervisor，虽然本次部署时在虚拟机上完成的，但是前面配置了KVM嵌套虚拟化，因此可以选择“KVM”

![选区_033](/img/blog_img/033.png)

        *   选择网络类型

![选区_034](/img/blog_img/034.png)

        *   选择后端存储，本次Cinder和Glance均使用Ceph存储

![选区_035](/img/blog_img/035.png)

        *   选择附加服务，可根据需求进行选择，本次测试不安装附加服务

![选区_036](/img/blog_img/036.png)

        *   完成配置

![选区_037](/img/blog_img/037.png)

![选区_038](/img/blog_img/038.png)

*   选择“网络”面板，配置网络

![选区_039](/img/blog_img/039.png)

![选区_040](/img/blog_img/040.png)

![选区_041](/img/blog_img/041.png)

![选区_042](/img/blog_img/042.png)

![选区_043](/img/blog_img/043.png)

![选区_047](/img/blog_img/047.png)

*   选择“设置”面板，配置其他OpenStack参数（此处未贴图部分使用默认值）

    *   此处“Public Key”为Fuel主机上root用户生成的ssh密钥对的公钥

    ![选区_045](/img/blog_img/045.png)

![选区_046](/img/blog_img/046.png)

![选区_047](/img/blog_img/047.png)

*   增加节点

    *   启动所有的“Controller”节点，等待一段时间后，可看到WEB界面右上角有发现三个新节点的提示，如下图所示：

![选区_049](/img/blog_img/049.png)

        *   此时，点击“增加节点”按钮，可看到新发现的三个新节点的信息，如下图所示：

![选区_050](/img/blog_img/050.png)

        *   在“分配角色”中选择为三个节点所分配的角色，同时勾选三个节点，点击“应用变更”，将三个节点配置为“controller”节点

![选区_051](/img/blog_img/051.png)

        *   启动所有的“Compute”节点，等待一段时间后，可看到WEB界面右上角有发现两个新节点的提示，如下图所示：

![选区_052](/img/blog_img/052.png)

        *   此时，点击“增加节点”按钮，可看到新发现的两个新节点的信息，如下图所示：

![选区_053](/img/blog_img/053.png)

        *   在“分配角色”中选择为两个节点所分配的角色，同时勾选两个节点，点击“应用变更”，将三个节点配置为“compute”节点

![选区_054](/img/blog_img/054.png)

        *   启动所有的“Ceph”节点，等待一段时间后，可看到WEB界面右上角有发现三个新节点的提示，如下图所示：

![选区_055](/img/blog_img/055.png)

        *   此时，点击“增加节点”按钮，可看到新发现的三个新节点的信息，如下图所示：

![选区_056](/img/blog_img/056.png)

        *   在“分配角色”中选择为三个节点所分配的角色，同时勾选三个节点，点击“应用变更”，将三个节点配置为“ceph”节点

![选区_057](/img/blog_img/057.png)
*   对新增加的节点进行网络配置

    *   选中新增加的所有节点，点击“网络配置”

![选区_058](/img/blog_img/058.png)

        *   按下图顺序配置网卡与网络的对应关系（此处与节点物理网卡与网络对应关系要匹配）

![选区_059](/img/blog_img/059.png)
*   验证网络配置

    *   切换到“网络”面板，点击“验证网络”，对整个环境的网络配置进行验证

![选区_060](/img/blog_img/060.png)

        *   网络验证成功，如下图所示

![选区_061](/img/blog_img/061.png)
*   开始自动化部署OpenStack环境

    *   点击“部署变更”按钮，开始自动化部署环境

![选区_062](/img/blog_img/062.png)

![选区_063](/img/blog_img/063.png)

        *   fuel首先会在所有节点安装CentOS6.5操作系统

![选区_064](/img/blog_img/064.png)

        *   然后安装/配置OpenStack相关组件

![选区_065](/img/blog_img/065.png)
*   部署成功，可看到成功提示

![选区_066](/img/blog_img/066.png)

![选区_067](/img/blog_img/067.png)

* * *

OK，到此使用Fuel部署OpenStack环境已经完成。

*   可以连接到Management网络，访问http://10.0.0.2来访问OpenStack Dashboard。

* * *

**『注 1』：** **KVM嵌套虚拟化**配置方法

*   确认系统当前是否配置了KVM嵌套虚拟化，如返回值为“Y”，表示已配置KVM嵌套虚拟化，不需要后续操作；如返回值为“N”，则需要按以下描述进行配置。

    # cat /sys/module/kvm_intel/parameters/nested
    `</pre>
*   卸载内核模块
    <pre>`# modprobe -r kvm-intel
    `</pre>
*   重新加载内核模块，同时将“nested”参数设置为“1”
    <pre>`# modprobe kvm-intel nested=1
    `</pre>
*   确认配置是否成功，如返回值为“Y”，表示配置成功
    <pre>`# cat /sys/module/kvm_intel/parameters/nested
    `</pre>
*   以上配置方法在系统重启后会丢失，如需使配置永久生效，需作如下操作
    <pre>`# echo "options kvm-intel nested=1" &gt; /etc/modprobe.d/kvm-intel.conf

* * *
