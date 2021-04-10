---
layout: post
title:  "基于Docker来存取redis数据"
date:   2017-07-08 23:49:08
categories: Redis
tags: Redis Docker
---

* content
{:toc}

### 1. Pull Redis Image

输入命令：docker pull redis，从Docker Hub上面拉取一份Redis p_w_picpath

<img width="80%" src="https://yuanzhitang.github.io/images/pull-redis-image.png"/>


### 2. 创建Redis 实例

输入命令：docker run -d --name mikeredis -p 6379 redis ， 启动 Redis并命名为mikeredis,port 采用默认端口

<img width="80%" src="https://yuanzhitang.github.io/images/create-redis-docker-container.png"/>

### 3.  连接Redis Server

输入命令：```shell docker run -it --link mikeredis:redis --rm redis redis-cli -h redis -p 6379```
并做简单的Set/Get Key 操作

<img width="80%" src="https://yuanzhitang.github.io/images/run-redis-container-image.png"/>

存取Key为name的数据成功！