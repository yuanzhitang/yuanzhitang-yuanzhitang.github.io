---
layout: post
title:  "Redis基础教程第5节 远程访问 Redis 基于 Python"
date:   2016-05-31 14:48:14
categories: Redis
tags: Redis
---

* content
{:toc}

本次Redis安装在Ubuntu Linux 虚拟机，基于VMPlayer。



### 1. 设置VM采用VMnet1

<img width="80%" src="https://yuanzhitang.github.io/images/set-vm-vmnet1.png"/>


### 2. 查看本机IP
<img width="80%" src="https://yuanzhitang.github.io/images/check-linux-ip.png"/>


### 3. 修改redis server的redis.conf使其支持外部IP

<img width="80%" src="https://yuanzhitang.github.io/images/change-redis-config-support-external-ip.png"/>



### 4. 启动Redis带有conf
<img width="80%" src="https://yuanzhitang.github.io/images/start-redis.png"/>


### 5. Python远程访问Redis server
<img width="80%" src="https://yuanzhitang.github.io/images/access-redis-via-python.png"/>



使用Redis Client连接Redis Server : 

```shell
tangym@ubuntu:~/redis-2.8.12/src$ 
redis-cli -h 192.168.162.128 -p 6379
```
