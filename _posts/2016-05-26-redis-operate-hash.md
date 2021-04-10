---
layout: post
title:  "Redis基础教程第4节 Hash"
date:   2016-05-26 10:26:58
categories: Redis
tags: Redis
author: Michael
mathjax: true
---

* content
{:toc}

Redis hash是一个string类型的field和value映射表。hash特别适合于存储对象。相对存成string，现在存为一个hash类型中可以占用更少的内存。且可以更方便的存储整个对象。



### hset
```shell
redis 127.0.0.1:6379> hset user:001 name michael
(integer) 1
redis 127.0.0.1:6379> hget user:001 name
"michael"
```
### hsetnx
```shell
redis 127.0.0.1:6379> hsetnx user:003 name jason
(integer) 1
redis 127.0.0.1:6379> hsetnx user:003 name tom
(integer) 0 -- set failed
redis 127.0.0.1:6379> hget user:003 name
"jason"  -- value is not changed.
```

### hmset/hmget
```shell
redis 127.0.0.1:6379> hmset user:004 name michael age 29 sex 1
OK
redis 127.0.0.1:6379> hget user:004 name
"michael"
redis 127.0.0.1:6379> hget user:004 age
"29"
redis 127.0.0.1:6379> hget user:004 sex
"1"
redis 127.0.0.1:6379> hmget user:004 name age sex
1) "michael"
2) "29"
3) "1"
```

### hincrby 
```shell
redis 127.0.0.1:6379> hincrby user:004 age 5
(integer) 34
redis 127.0.0.1:6379> hget user:004 age
"34"
```

### hexists 
```shell
redis 127.0.0.1:6379> hexists user:004 age
(integer) 1
redis 127.0.0.1:6379> hexists user:004 address
(integer) 0

```
### hlen 
```shell
redis 127.0.0.1:6379> hlen user:004
(integer) 3
```

### hdel 
```shell
redis 127.0.0.1:6379> hdel user:004 age
(integer) 1
redis 127.0.0.1:6379> hexists user:004 address
(integer) 0
redis 127.0.0.1:6379> hget user:004 age
(nil)
```

### hkeys hvals hgetall 
```shell
redis 127.0.0.1:6379> hkeys user:004
1) "name"
2) "sex"
redis 127.0.0.1:6379> hvals user:004
1) "michael"
2) "1"
redis 127.0.0.1:6379> hgetall user:004
1) "name"
2) "michael"
3) "sex"
4) "1"
```