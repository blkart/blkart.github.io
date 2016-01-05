---
layout: post
title: 『systemd』使用技巧
date: 2015-06-08
categories: blog
tags: [linux,systemd]
description: 『systemd』使用技巧

---

systemctl可以管理远端服务器上的服务

    # systemctl --help
    -H --host=[USER@]HOST
                      Show information for remote host

示例：

    # systemctl -H node-4 status ntpd
    Warning: Permanently added 'node-4' (ECDSA) to the list of known hosts.
    ntpd.service - Network Time Service
        Loaded: loaded (/usr/lib/systemd/system/ntpd.service; enabled)
        Active: active (running) since 五 2015-06-05 20:07:16 CST; 2 days ago
     Main PID: 13192
        CGroup: /system.slice/ntpd.servicee
    
