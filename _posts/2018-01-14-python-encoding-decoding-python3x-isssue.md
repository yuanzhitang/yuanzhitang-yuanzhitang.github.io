---
layout: post
title:  "Python3.X Socket 一个编码与解码的坑"
date:   2018-01-14 17:04:41
categories: Python
tags: Python requests BeautifulSoup Spider
author: Michael
mathjax: true
---

* content
{:toc}

在看《Python核心编程》第三版 讲述网络编程Socket的知识，在练习中采用Python 3 的代码中遇到一个与编码解码有关的坑，本文将给予详细的介绍。

### 软件环境
Python: 3.6.0
库: socket

## 1. 问题初见
仿照书中的代码（中文版 55-56页） 加上自己的一点改动在我的环境中不能运行，总是报这个错误：TypeError: a bytes-like object is required, not 'str'

这里是我的客户端Socket代码
```python
from socket import *
from time import ctime

HOST = 'localhost'
PORT = 10001
ADDRESS = (HOST, PORT)

clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect(ADDRESS)

while True:
    data = input('请输入消息:')
    if not data:
        break

    clientSocket.send(data)
    data = clientSocket.recv(1024)
    if not data:
        break

    print("服务器返回的消息是:", data.decode('utf-8'))

clientSocket.close()
```

我的环境是: Python 3.6.0， 怎么破？

## 2. 研究错误 TypeError: a bytes-like object is required, not 'str'
错误的位置是在代码clientSocket.send(data)部分，但是翻看python socket .send()源代码_socket.py 方法说明
```python
def send(self, data, flags=None): # real signature unknown; restored from __doc__

    send(data[, flags]) -> count

    Send a data string to the socket.  For the optional flags
    argument, see the Unix manual.  Return the number of bytes
    sent; this may be less than len(data) if the network is busy.

    pass
```
这个send方法的参数期望的是一个 "a data string" 啊，而我确实给了一个string。

哪里出问题了？ 继续查看官方文档Socket，发现原因了。

官方对Socket的说明：
```python
socket.send(bytes[, flags])
```
可以看到在Python 3中send()方法期望的是一个bytes, 而不是str
看来我我前面看到的是假的源代码参数的说明。哈哈。

## 3. 用encode() 方法解决客户端Socket 发送错误
解决错误的方法就是在调用send()方法之前对字符串类型数据进行encode，将字符串转化成bytes
代码如下：
```python
clientSocket.send(data.encode())
```
与此同时，在服务端运行的时候也遇到了类似数据无法接收的问题。
如下代码得到的data，是无法直接打印的。
```python
data = clientSocket.recv(1024)
```p
如果要打印data数据的话，也要调用decode()从而将数据从bytes转化为str。

## 4. encode() 和 decode()
encode()编码 ： str -> bytes
decode()解码 ： bytes -> str

默认的encoding是 utf-8

更多内容见官方文档：
str.encode()
bytes.decode()

## 5. 完整Socket代码
### 服务端：
```python
from socket import *
from time import ctime

HOST = 'localhost'
PORT = 10001
ADDRESS = (HOST, PORT)

serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(ADDRESS)
serverSocket.listen(5)

while True:
    print("等待客户端连接...")
    clientSocket, address = serverSocket.accept()
    print(address, "已经成功连接至本服务器")

    while True:
        data = clientSocket.recv(1024)
        if not data:
            break

        replyMsg = data.decode() + "[" + ctime() + ']'
        clientSocket.send(replyMsg.encode())

    clientSocket.close()
serverSocket.close()
```
### 客户端：
```python
from socket import *
from time import ctime

HOST = 'localhost'
PORT = 10001
ADDRESS = (HOST, PORT)

clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect(ADDRESS)

while True:
    data = input('请输入消息:')
    if not data:
        break

    clientSocket.send(data.encode())
    data = clientSocket.recv(1024)
    if not data:
        break

    print("服务器返回的消息是:", data.decode('utf-8'))

clientSocket.close()
```
《Python核心编程》第三版原始代码P55-56在Python3中并不能运行的问题，算不算一个错误呢？ 欢迎大家交流 ！