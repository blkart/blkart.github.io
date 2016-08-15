---
layout: post
title: 使用 cgroups 控制系统资源
date: 2016-08-15
categories: blog
tags: [cgroups]
description: 使用 cgroups 控制系统资源

---

简单记录几个使用 cgroups 控制系统资源的例子

> 测试环境：CentOS 7

### 管理进程 CPU 资源

* 耗 CPU 资源的测试脚本

```
#!/bin/bash
# filename: test_cpu.sh

x=0
while [ True ];do
    x=$x+1
done
```

* 运行脚本，查看对应进程 CPU 使用情况

```
# top
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND         
19235 root      20   0  113272   1672   1232 R  98.2  0.0   0:11.88 test_cpu.sh
```

> 从以上输出信息中可以看到，脚本运行对应的进程 CPU 使用率已经接近 100%

* 使用 cgroups 限制这个进程对 CPU 资源的使用

```
# mkdir /sys/fs/cgroup/cpu/test_cpu/
# echo 50000 > /sys/fs/cgroup/cpu/test_cpu/cpu.cfs_quota_us
# echo 19235 > /sys/fs/cgroup/cpu/test_cpu/tasks
```

* 再次查看脚本运行对应的进程 CPU 使用率

```
# top
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND         
19235 root      20   0  114788   2932   1232 R  49.8  0.1   5:34.82 test_cpu.sh
```

### 管理进程 MEM 资源


```
#!/bin/bash
# filename: test_mem.sh

x='a'
while [ True ];do
    x=$x$x
done
```

* 运行脚本，查看对应进程 MEM 使用情况

```
# top
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND         
26459 root      20   0 3258888 1.430g   1236 R  33.8 37.1   0:45.42 test_mem.sh
```

> 从以上输出信息中可以看到，脚本运行对应的进程 MEM 使用率已经打到 37.1%

* 使用 cgroups 限制这个进程对 MEM 资源的使用

```
# mkdir /sys/fs/cgroup/memory/test_mem
# echo 1048576 > /sys/fs/cgroup/memory/test_mem/memory.limit_in_bytes
# echo 26459 > /sys/fs/cgroup/memory/test_mem/tasks
```

* 再次查看脚本运行对应的进程 MEM 使用率

```
# top
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND         
26459 root      20   0 3258888 1.430g   1236 D  33.8 37.1   0:45.42 test_mem.sh
```

### 管理进程 IO 资源

* 执行消耗 IO 的命令，查看对应进程 IO 资源的使用情况

```
dd if=/dev/sda of=/dev/null
```

```
# iotop
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                       
28103 be/4 root      190.80 M/s    0.00 B/s  0.00 % 35.94 % dd if=/dev/vda of=/dev/null
```

* 使用 cgroups 限制这个进程对 IO 资源的使用

```
# mkdir /sys/fs/cgroup/blkio/test_blkio
# ls -l /dev/vda
brw-rw---- 1 root disk 252, 0 8月   3 16:34 /dev/vda
# echo '252:0 1048576' > /sys/fs/cgroup/blkio/test_blkio/blkio.throttle.read_bps_device
# echo 28103 > /sys/fs/cgroup/blkio/test_blkio/tasks
```

* 再次查看命令运行对应的进程 IO 资源的使用情况

```
# iotop
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                 
28103 be/4 root     1880.24 K/s    0.00 B/s  0.00 % 99.99 % dd if=/dev/vda of=/dev/null
```
