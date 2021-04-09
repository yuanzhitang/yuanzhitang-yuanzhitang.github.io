---
layout: post
title:  "Redis基础教程第3节 Strings"
date:   2016-05-25 17:00:11
categories: Redis
tags: Redis
---

* content
{:toc}

tangym@ubuntu:~$ cd redis-2.8.12

tangym@ubuntu:~/redis-2.8.12$ cd src

tangym@ubuntu:~/redis-2.8.12/src$ redis-cli



### Set
```
redis 127.0.0.1:6379> get name
"lijie"
redis 127.0.0.1:6379> set name michael
OK
```

### Get
```
redis 127.0.0.1:6379> get name
"michael"
redis 127.0.0.1:6379> setnx name micahel
(integer) 0
```

### Setnx
```
redis 127.0.0.1:6379> setnx age 10
(integer) 1
redis 127.0.0.1:6379> setex date 10 2010
OK
redis 127.0.0.1:6379> get date
"2010"
redis 127.0.0.1:6379> get date
(nil)
redis 127.0.0.1:6379> set email michael@unisys.com
OK
redis 127.0.0.1:6379> get email
"michael@unisys.com"
```

### setrange
```
redis 127.0.0.1:6379> setrange email 8 qq.com
(integer) 18
redis 127.0.0.1:6379> get email
"michael@qq.com.com"
```

### mset
```
redis 127.0.0.1:6379> mset key1 m1 key2 m2
OK
redis 127.0.0.1:6379> get key1
"m1"
redis 127.0.0.1:6379> get key2
"m2"
redis 127.0.0.1:6379> get key2
"m2"
redis 127.0.0.1:6379> get key3
(nil)
```

### msetnx
```
redis 127.0.0.1:6379> msetnx key4 unisys.com key5 jasson.net key3 m3 key2 m2
(integer) 0
redis 127.0.0.1:6379> get key4
(nil)
redis 127.0.0.1:6379> msetnx key4 unisys.com key5 jasson.net key3 m3
(integer) 1
redis 127.0.0.1:6379> get key1
"m1"
```

### getrange
```
redis 127.0.0.1:6379> getrange key1 0 1
"m1"
redis 127.0.0.1:6379> getrange key1 0 0
"m"
```

### getset
```
redis 127.0.0.1:6379> getset key1 20
"m1"
```

### mget
```
redis 127.0.0.1:6379> mget key1 key2 key3 key4 key5
1) "20"
2) "m2"
3) "m3"
4) "unisys.com"
5) "jasson.net"
```

### incr 
```
redis 127.0.0.1:6379> incr key6
(integer) 101
redis 127.0.0.1:6379> incr key6
(integer) 102
redis 127.0.0.1:6379> get key6
"102"
```

### incrby 
```
redis 127.0.0.1:6379> incrby key6 1
(integer) 103
redis 127.0.0.1:6379> incrby key6 2
(integer) 105
redis 127.0.0.1:6379> incrby key6 -2
(integer) 103
```

### decr/decrby 
```
redis 127.0.0.1:6379> decr key6 1
(error) ERR wrong number of arguments for 'decr' command
redis 127.0.0.1:6379> decr key6
(integer) 102
redis 127.0.0.1:6379> decrby key6 1
(integer) 101
```

### append 
```
redis 127.0.0.1:6379> get name
"michael"
redis 127.0.0.1:6379> append name .net
(integer) 11
redis 127.0.0.1:6379> get name
"michael.net"
```

### strlen 
```
redis 127.0.0.1:6379> strlen name
(integer) 11
```