---
layout: post
title:  "Redis基础教程第6节 List"
date:   2016-06-15 10:26:58
categories: Redis
tags: Redis
author: Michael
mathjax: true
---

* content
{:toc}

list是一个内部采用双向链表(double linked list) 结构，像列表两端添加元素的时间复杂度为O(1)。主要功能是push、pop、获取一个范围的所有值等，操作中key理解为链表的名字。

链表的最大长度是(2的32次方)。我们可以通过push,pop操作从链表的头部或者尾部添加删除元素。这使得list既可以用作栈，也可以用作队列。



list的pop操作均有阻塞版本的，当我们[lr]pop一个list对象时，如果list是空，或者不存在，会立即返回nil。但是阻塞版本的b[lr]pop可以则可以阻塞，当然可以加超时时间，超时后也会返回nil。为什么要阻塞版本的pop呢，主要是为了避免轮询。



举个简单的例子如果我们用list来实现一个工作队列。执行任务的thread可以调用阻塞版本的pop去获取任务这样就可以避免轮询去检查是否有任务存在。当任务来时候工作线程可以立即返回，也可以避免轮询带来的延迟。



LPUSH Key value 左边插入

RPUSH Key value 右边插入

LPop key 左边弹出

RPop key 右边弹出

BLPOP,BRPOP阻塞式左/右弹出



### lpush
```shell
129.223.248.154:6379> lpush members ben
(integer) 1
129.223.248.154:6379> lpush members jeff
(integer) 2
129.223.248.154:6379> lpush members mike jeme
(integer) 6
```

### lpop
```shell
129.223.248.154:6379> lpop members
"raymond"
129.223.248.154:6379> rpop members
"ben"
```

### llen
```shell
129.223.248.154:6379> llen members
(integer) 4
```

### lrange
(lrange firstqueue 0 -1 列出list中全部元素值）
```shell
129.223.248.154:6379> lrange members 0 2
1) "richard"
2) "jemery"
3) "mike"
129.223.248.154:6379> llen members
(integer) 4
129.223.248.154:6379> lrange members 0 3
1) "richard"
2) "jemery"
3) "mike"
4) "jeff"
129.223.248.154:6379> lrange members 0 4
1) "richard"
2) "jemery"
3) "mike"
4) "jeff"
129.223.248.154:6379> lrange members 0 -1
1) "richard"
2) "jemery"
3) "mike"
4) "jeff"
5) "derek"
```

### rpop
```shell
129.223.248.154:6379> rpop members
"derek"
129.223.248.154:6379> lpop members
"richard"
129.223.248.154:6379> lrange members 0 -1
1) "jemery"
2) "mike"
3) "jeff"
```

### lindex
```shell
129.223.248.154:6379> lindex members 1
"mike"
129.223.248.154:6379> llen members
(integer) 3
129.223.248.154:6379> rpush firstqueue 3 2 1
(integer) 3
129.223.248.154:6379> lrange firstqueue 0 -1
1) "3"
2) "2"
3) "1"
129.223.248.154:6379> lpush secqueue 3 2
(integer) 2
129.223.248.154:6379> lrange secqueue 0 -1
1) "2"
2) "3"
```

### rpoplpush
从第一个list的尾部移除元素并添加到第二个list的头部,最后返回被移除的元素值，整个操作是原子的.如果第一个list是空或者不存在返回nil
```shell
129.223.248.154:6379> rpoplpush firstqueue secqueue
"1"
129.223.248.154:6379> lrange firstqueue 0 -1
1) "3"
2) "2"
129.223.248.154:6379> lrange secqueue 0 -1
1) "1"
2) "2"
3) "3"
```
