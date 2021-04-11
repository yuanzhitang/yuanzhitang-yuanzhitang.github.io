---
layout: post
title:  "在CentOS 7最小化服务器版本中如何启用“ifconfig”命令？"
date:   2018-01-01 23:00:37
categories: Linux
tags: CentOS Linux
author: Michael
mathjax: true
---

* content
{:toc}

最近安装了一个最小化版本的CentOS 7，发现不能使用 ifconfig命令，界面提示：
```shellifconfig command not found.```



### 解决办法

从yum找出哪个包括ifconfig 命令
```shell
[root@tangym ~]# yum provides ifconfig Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile

base: mirrors.zju.edu.cn
extras: mirrors.163.com
updates: mirrors.163.com
net-tools-2.0-0.22.20131004git.el7.x86_64 : Basic networking tools
Repo : @base
Matched from:
Filename : /usr/sbin/ifconfig
```
从上面可以看到 net-tools 包中可以找到ifconfig命令
安装net-tools
```shell
[root@tangym ~]# yum provides ifconfig
```
安装完成之后，将可以使用ifconfig 命令了

```shell
[root@tangym ~]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
inet 192.168.1.20 netmask 255.255.255.0 broadcast 192.168.1.255
inet6 fe80::1157:71fe:de9b:3e7e prefixlen 64 scopeid 0x20<link>
ether 00:15:5d:01:07:3c txqueuelen 1000 (Ethernet)
RX packets 29890 bytes 42966542 (40.9 MiB)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 18814 bytes 1278145 (1.2 MiB)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

        lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
                        inet 127.0.0.1  netmask 255.0.0.0
                        inet6 ::1  prefixlen 128  scopeid 0x10<host>
                        loop  txqueuelen 1  (Local Loopback)
                        RX packets 0  bytes 0 (0.0 B)
                        RX errors 0  dropped 0  overruns 0  frame 0
                        TX packets 0  bytes 0 (0.0 B)
                        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```