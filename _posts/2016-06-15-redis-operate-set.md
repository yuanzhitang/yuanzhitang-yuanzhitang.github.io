---
layout: post
title:  "Redis基础教程第7节 - Set"
date:   2016-06-15 17:20:58
categories: Redis
tags: Redis
author: Michael
mathjax: true
---

* content
{:toc}

Set类型是一个没有排序的集合，可以在该类型那个执行添加、删除或判断某一元素是否存在等操作。由于Redis的内部是使用值为空的hash table实现的，所以操作的时间复杂度为O(1), 最多存储2^32-1个字符串。



Set集合中不允许出现重复的元素，和List类型相比，Set类型在功能上另一个优势是可以进行并集，交集，差集运算。



### sadd
```shell
129.223.248.154:6379> sadd students tim
(integer) 1
129.223.248.154:6379> sadd students tim ben
(integer) 1
```

### srem
```shell
129.223.248.154:6379> srem student tim
(integer) 0
129.223.248.154:6379> srem students tim
(integer) 1
```

### smembers、sismember
```shell
129.223.248.154:6379> smembers students
1) "ben"
129.223.248.154:6379> sismember students ben
(integer) 1
129.223.248.154:6379> sismember students tim
(integer) 0
```

### sdiff 差集
```shell
129.223.248.154:6379> sadd setDiffA 1 2 3
(integer) 3
129.223.248.154:6379> sadd setDiffB 2 3 4
(integer) 3
129.223.248.154:6379> 
sdiff 
setDiffA setDiffB
1) "1"
```

### sinter 交集
```shell
129.223.248.154:6379> sadd setInterA 1 2 3
(integer) 3
129.223.248.154:6379> sadd setInterB 2 3 4
(integer) 3
129.223.248.154:6379> 
sinter 
setInterA setInterB
1) "2"
2) "3"
```

### sunion 并集
```shell
129.223.248.154:6379> sadd setUnionA 1 2 3
(integer) 3
129.223.248.154:6379> sadd setUnionB 2 4 6
(integer) 3
129.223.248.154:6379> 
sunion 
setUnionA setUnionB
1) "1"
2) "2"
3) "3"
4) "4"
5) "6"
```

### scard 集合总数
```shell
129.223.248.154:6379> 
scard 
students
(integer) 1
129.223.248.154:6379> smembers students
1) "ben"
129.223.248.154:6379> srandmember students
"ben"
129.223.248.154:6379> sadd students mike
(integer) 1
129.223.248.154:6379> srandmember students
"ben"
129.223.248.154:6379> srandmember students
"mike"
129.223.248.154:6379> srandmember students
"mike"
129.223.248.154:6379> sadd memebrs a b c
(integer) 3
129.223.248.154:6379> srandmember students 2
1) "ben"
2) "mike"
```

### srandmember
```shell
129.223.248.154:6379> sadd letters a b c
(integer) 3
129.223.248.154:6379> 
srandmember 
letters 2
1) "b"
2) "c"
```

### spop
```shell
129.223.248.154:6379> spop letters
"b"
129.223.248.154:6379> smembers letters
1) "a"
2) "c"
129.223.248.154:6379>
```