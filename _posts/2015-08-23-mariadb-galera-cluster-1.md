---
layout: post
title: MariaDB Galera Cluster —— 环境搭建
date: 2015-08-23
categories: blog
tags: [MariaDB]
description: MariaDB Galera Cluster —— 环境搭建

---

## 简介

Galera Cluster 是一个同步多主数据库集群，可以使得MariDB的所有节点保持同步，Galera为MariaDB提供了同步复制（相对于原生的异步复制）,因此其可以保证HA。在使用InnoDB存储引擎时，还可以支持多主节点功能，这意味着Galera Cluster支持对任何节点的读写操作。

## 特点

* 真正的多主架构，任何节点都可以进行读写
* 同步复制，各节点间无延迟且节点宕机不会导致数据丢失
* 紧密耦合，所有节点均保持相同状态，节点间无不同数据
* 无需主从切换操作或使用VIP
* 热Standby，在Failover过程中无停机时间（由于不需要Failover）
* 自动节点配置，无需手工备份当前数据库并拷贝至新节点
* 支持InnoDB存储引擎
* 对应于透明，无需更改应用或是进行极小的更改
* 无需进行读写分离

## 环境搭建

### 环境准备

#### 三台服务器（操作系统CentOS 7.0）

<table>
<thead>
<tr>
  <th>主机名</th>
  <th>IP</th>
</tr>
</thead>
<tbody>
<tr>
  <td>node-1</td>
  <td>192&#46;168.2.21</td>
</tr>
<tr>
  <td>node-2</td>
  <td>192&#46;168.2.22</td>
</tr>
<tr>
  <td>node-3</td>
  <td>192&#46;168.2.23</td>
</tr>
</tbody>
</table>

#### 修改SELinux模式

    # sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/sysconfig/selinux
    # setenforce 0

#### 关闭防火墙

    # systemctl stop firewalld

#### 卸载Postfix

> 安装mysql-wsrep-server包时可能由于Postfix存在导致安装失败，所以先卸载掉该软件包，如系统中需要Postfix提供邮件服务，可以在Galera Cluster部署完成后，重新安装Postfix即可。

    # yum remove postfix

### 集群部署（以下操作在三个节点重复执行）

#### 配置YUM源

    # vim /etc/yum.repos.d/mariadb.repo
    [mariadb]
    name = MariaDB
    baseurl = http://yum.mariadb.org/5.5/centos7-amd64
    gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
    gpgcheck=1

#### 安装软件包

    # yum -y install MariaDB-client MariaDB-Galera-server galera

#### 修改配置文件

    # mv /etc/my.cnf{,.bak}
    # vim /etc/my.cnf
    [mysqld]
    datadir=/var/lib/mysql
    socket=/var/lib/mysql/mysql.sock
    user=mysql
    # Ensure that the binary log format is set to use row-level replication, as opposed to statement-level replication.
    binlog_format=ROW
    bind-address=0.0.0.0
    # Ensure that the default storage engine is InnoDB
    default_storage_engine=innodb
    innodb_autoinc_lock_mode=2
    innodb_flush_log_at_trx_commit=0
    innodb_buffer_pool_size=122M
    wsrep_provider=/usr/lib64/galera/libgalera_smm.so
    wsrep_provider_options="gcache.size=300M; gcache.page_size=1G"
    wsrep_cluster_name="blkart_cluster"
    wsrep_cluster_address="gcomm://192.168.2.21,192.168.2.22,192.168.2.23"
    wsrep_node_name="node-1"
    wsrep_node_address="192.168.2.21"
    wsrep_sst_method=rsync
    
    [mysql_safe]
    log-error=/var/log/mysqld.log
    pid-file=/var/run/mysqld/mysqld.pid

#### 配置swap空间

    # fallocate -l 512M /swapfile
    # dd if=/dev/zero of=/swapfile bs=1M count=512
    # chmod 600 /swapfile
    # ll -a / | grep swapfile
    -rw-------.   1 root root 536870912 7月  28 16:07 swapfile
    # mkswap /swapfile
    # swapon /swapfile
    # vim /etc/fstab
    /swapfile none swap defaults 0 0
    # swapon --summary
    Filename        Type        Size     Used    Priority
    /swapfile       file        524284   0       -1

### 集群初始化

#### 启动第一个节点

    # /etc/init.d/mysql start --wsrep-new-cluster

> 注意： '--wsrep-new-cluster'选项只在第一次启动第一个节点时使用！

#### 确认数据库服务启动成功

    # mysql -u root -p
    Enter password: 
    Welcome to the MariaDB monitor.  Commands end with ; or g.
    Your MariaDB connection id is 9
    Server version: 5.5.44-MariaDB-wsrep MariaDB Server, wsrep_25.11.r4026
    
    Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.
    
    Type 'help;' or 'h' for help. Type 'c' to clear the current input statement.
    
    MariaDB [(none)]> SHOW STATUS LIKE 'wsrep_cluster_size';
    +--------------------+-------+
    | Variable_name        | Value |
    +--------------------+-------+
    | wsrep_cluster_size    | 1     |
    +--------------------+-------+
    1 row in set (0.00 sec)

> 注意： 此时不要重启mysql服务!

#### 将其他节点加入集群（在第二及第三个节点执行以下命令）

    # systemctl start mysql
    # mysql -u root -p
    Enter password: 
    Welcome to the MariaDB monitor.  Commands end with ; or g.
    Your MariaDB connection id is 9
    Server version: 5.5.44-MariaDB-wsrep MariaDB Server, wsrep_25.11.r4026
    
    Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.
    
    Type 'help;' or 'h' for help. Type 'c' to clear the current input statement.
    
    MariaDB [(none)]> SHOW STATUS LIKE 'wsrep_cluster_size';
    +--------------------+-------+
    | Variable_name        | Value |
    +--------------------+-------+
    | wsrep_cluster_size    | 3     |
    +--------------------+-------+
    1 row in set (0.00 sec)

### 集群测试

#### 复制测试

在node-1连接到数据库

    # mysql -u root -p
    Enter password: 
    Welcome to the MariaDB monitor.  Commands end with ; or g.
    Your MariaDB connection id is 9
    Server version: 5.5.44-MariaDB-wsrep MariaDB Server, wsrep_25.11.r4026
    
    Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.
    
    Type 'help;' or 'h' for help. Type 'c' to clear the current input statement.
    
    MariaDB [(none)]> SHOW STATUS LIKE 'wsrep_%';
    
     +---------------------------+------------+
     | Variable_name                | Value      |
     +---------------------------+------------+
     ...
     | wsrep_local_state_comment  | Synced (6) |
     | wsrep_cluster_size           | 3           |
     | wsrep_ready                | ON          |
     +---------------------------+------------+
    MariaDB [(none)]> CREATE DATABASE galeratest;
    MariaDB [(none)]> USE galeratest;
    MariaDB [(none)]> CREATE TABLE test_table (
        -> id INT PRIMARY KEY AUTO_INCREMENT,
        -> msg TEXT ) ENGINE=InnoDB;
    MariaDB [(none)]> INSERT INTO test_table (msg)
        -> VALUES ("Hello my dear cluster.");
    MariaDB [(none)]> INSERT INTO test_table (msg)
        -> VALUES ("Hello, again, cluster dear.");

在node-2连接到数据库

    # mysql -u root -p
    Enter password: 
    Welcome to the MariaDB monitor.  Commands end with ; or g.
    Your MariaDB connection id is 9
    Server version: 5.5.44-MariaDB-wsrep MariaDB Server, wsrep_25.11.r4026
    
    Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.
    
    Type 'help;' or 'h' for help. Type 'c' to clear the current input statement.
    
    MariaDB [(none)]> USE galeratest;
    MariaDB [(none)]> SELECT * FROM test_table;
    
     +----+-----------------------------+
     | id | msg                         |
     +----+-----------------------------+
     |  1 | Hello my dear cluster.      |
     |  2 | Hello, again, cluster dear. |
     +----+-----------------------------+
