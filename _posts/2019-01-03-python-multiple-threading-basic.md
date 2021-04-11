---
layout: post
title:  "Python 3多线程编程学习笔记-基础篇"
date:   2019-01-03 10:45:42
categories: Python
tags: Python 多线程
author: Michael
mathjax: true
---

* content
{:toc}

本文是学习《Python核心编程》的学习笔记，介绍了Python中的全局解释器锁和常用的两个线程模块：thread, threading，并对比他们的优缺点和给出简单的列子。



## 全局解释器锁（GIL)
Python代码的执行都是有Python虚拟机进行控制的。当初设计Python的时候，考虑在主循环中只能有一个控制线程在执行，就像单核CPU进行多线程编程一样。
怎么做到这样控制的呢？就是这里的GIL来控制的，这个锁用来保证同时只有一个线程在运行。

执行方式：
<img width="100%" src="https://yuanzhitang.github.io/images/python-gil.png"/>
Python 3多线程编程学习笔记-基础篇

这几个细节知识点：

当调用外部代码（C/C++扩展的内置函数）时， GIL会保持锁定，知道函数执行结束
对于面向I/O的Python例程，GIL会在I/O调用前被释放，从而允许其他线程在I/O执行期间运行
如果是针对计算密集型的操作代码，该线程整个时间片内更倾向于始终占有处理器和GIL。
所以，针对Python虚拟机单线程(GIL)的设计原因，只有线程在执行I/O密集型的应用，才能更好的发挥Python的并发性。

## 对比thread,threading
Python多个模块可以进行多线程编程，包括：thread,threading等。他们都可以用来创建和管理线程。
thread模块提供了基本的线程和锁定支持，而threading模块提供了更高级别，功能更全面的线程管理。

推荐使用更高级别的threading模块，下面是一个简单的对比：

特征要素	thread	threading
功能全面性	基本（偏底层）	全面，高级
守护进程	不支持	支持
线程同步原语	仅1个（Lock)	很多（Lock,Semaphore,Barrier...)
守护进程讲解
守护进程一般是一个等待客户端请求服务的服务器。如果没有客户端请求，守护进程一般是空闲的。一般把一个线程设置成守护进程，就表示这个线程是不重要的。所以进程退出时是不需要等待这个守护线程完成的。

但是原先的thread 模块是不区分守护或者非守护进程的，也就是说当主线程退出的时候，所有子线程都将终止，而不管他们是否仍在工作。如果你不想这种情况发生，那么就可以采用threading模块。整个Python程序（一般为主线程）将在所有非守护进程退出是才会退出。

设置守护进程，在线程启动之前设置：
thread.daemon = True

## 多线程实践
### threading实例
#### 方式1：创建一个thread实例，传递它一个函数
```python
import threading
from time import sleep, ctime

sleep_times = [4, 2]

def loop(threadNo, sleep_time):
    print('Start loop', threadNo, 'at:', ctime())
    sleep(sleep_time)    #Sleep一段时间
    print('loop', threadNo, 'done at:', ctime())

def main():
    print('starting at:', ctime())
    threads = []
    threadIds = range(len(sleep_times))

    for i in threadIds:
        thread = threading.Thread(target=loop, args=(i,sleep_times[i]))
        threads.append(thread)

    for t in threads:
        # 依次启动线程
        t.start()

    for t in threads:
        # 等待所有线程完成
        t.join() #将等待线程结束

    print('all Done at :', ctime())

if __name__ == '__main__':
    main()
```
#### 方式2：派生Thread的子类，并创建子类的实例
```python
import threading
from time import sleep, ctime

sleep_times = [4, 2]

class MyThread(threading.Thread):
    def __init__(self, func, args, name=''):
        threading.Thread.__init__(self)
        self.func = func
        self.name = name
        self.args = args

    def run(self):
        self.func(*self.args)

def loop(threadNo, sleep_time):
    print('Start loop', threadNo, 'at:', ctime())
    sleep(sleep_time)  # Sleep一段时间
    print('loop', threadNo, 'done at:', ctime())

def main():
    print('starting at:', ctime())
    threads = []  # 用于存储所有线程实例的列表
    threadIds = range(len(sleep_times))

    for i in threadIds:
        # 创建线程实例
        thread = MyThread(loop, (i, sleep_times[i]))
        threads.append(thread)

    for t in threads:
        # 依次启动线程
        t.start()

    for t in threads:
        # 等待所有线程完成
        t.join()  # 将等待线程结束

    print('all Done at :', ctime())

if __name__ == '__main__':
    main()
```

#### thread实例
由于本人使用的python3.6,这个thread已经变成_thread
```python
import _thread
from time import sleep, ctime

sleep_times = [4, 2]

def loop(threadNo, sleep_time, lock):
    print('Start loop', threadNo, 'at:', ctime())
    sleep(sleep_time)    #Sleep一段时间
    print('loop', threadNo, 'done at:', ctime())
    lock.release()       #释放锁

def main():
    print('starting at:', ctime())
    locks = []
    threadIds = range(len(sleep_times))

    for i in threadIds:

        #通过调用_thread.allocate_lock获得锁对象
        lock = _thread.allocate_lock()

        #通过acquire()方法取得锁
        lock.acquire()
        locks.append(lock)

    for i in threadIds:
        # 依次启动线程
        _thread.start_new_thread(loop, (i, sleep_times[i], locks[i]))

    for i in threadIds:
        # 如果当前的锁Lock没有释放的话，一直循环等待
        while locks[i].locked():
            pass
    print('all Done at :', ctime())

if __name__ == '__main__':
    main()
```