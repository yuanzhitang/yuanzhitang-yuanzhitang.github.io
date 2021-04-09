---
layout: post
title:  "Redis基础教程第1节 Ubuntu Linux 上安装Redis"
date:   2016-05-23 22:41:34
categories: Redis
tags: Redis
---

* content
{:toc}

本文将介绍如何安装redis在ubuntu系统上面。



步骤如下：

1. ~$ sudo apt-get update

2. ~$ sudo apt-get install make gcc python-dev

3.  下载, 解压和编译compile Redis:
```
$ wget http://download.redis.io/releases/redis-3.2.0.tar.gz
    $ tar xzf redis-3.2.0.tar.gz
    $ cd redis-3.2.0
    $ make
```
4. 启动Redis-server:
    The binaries that are now compiled are available in the src directory. Run Redis with:d
```
michael@ubuntu:~$ cd redis-3.2.0
    $ src/redis-server
```
5. 启动默认Redis-client并与Server进行交互
```
 michael@ubuntu:~$ cd redis-3.2.0
    $ src/redis-cli
    redis> set name HelloWorld
    OK
    redis> get name
    "HelloWorld"
```