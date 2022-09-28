---
layout: post
title: ".NET 内存分配实例篇"
date: 2022-08-21 14:24:51
categories: .NET
tags: C# .NET GC Stack Heap
author: Michael
mathjax: true
---

* content
{:toc}

C#中类型主要分为引用类型和值类型，通常情况下值类型分配在堆栈（Stack）上，而引用类型则分配在托管堆(Heap）上。

下图列出了C#中的值类型和引用类型。如果想知道一个变量是值类型还是引用类型，可以采用`variable.GetType().IsValueType`去检测。




<img width="100%" src="https://yuanzhitang.github.io/images/csharp-types.png"/>



下面通过几个小例子演示内存的分配
### Int
Int类型占用内存根据所在的作用域进行出栈和入栈。比如下面的代码。num1先行拿到值10位于stack上，之后因为跳出作用域，值10将会进行出栈，之后一次分配20，30给num2和num3。此时在尝试访问num1是不行的，因为值10已经出栈不存在了
<img width="100%" src="https://yuanzhitang.github.io/images/memory-alloc-int.png"/>

在看下面这个代码
```cs
int num = 10;

int secNum = num;

secNum = 20;

Console.WriteLine($"{num},{secNum}");

```
首先分配10给num，这个时候将num赋值给secNum时，其实是将值复制了一份给secNum，也就是说在stack中，有两个10。 如果此时将secNum修改为20，并不会影响到num。所有最后的输出是 ```cs 10,20```
### 垃圾回收
对象的分配与回收流程是：当创建一个新对象是，对象将分配值Gen0，如果Gen0剩余空间不够，那么对将Gen0进行内存回收，Gen0上所在对象将会移至Gen1，如果此时Gen1剩余空间不够存放来自于Gen0的对象，那么将对Gen1进行垃圾回收，将Gen1的对象移动至Gen2。如果此时Gen2剩余空间也不够，那么将会对Gen2进行回收，这个又称为Full GC, 同时也会对大对象进行回收. 
<img width="100%" src="https://yuanzhitang.github.io/images/gc-workflow.png"/>

比如：现在有10个对象，其中对象2，3，5，6和10拥有Finalize方法，整个GC将经过如下几个阶段
- 初始
<img width="100%" src="https://yuanzhitang.github.io/images/gc-sample-init.png"/>
- 标记
<img width="100%" src="https://yuanzhitang.github.io/images/gc-sample-mark.png"/>
- 清理
<img width="100%" src="https://yuanzhitang.github.io/images/gc-sample-sweep.png"/>
- 压缩
<img width="100%" src="https://yuanzhitang.github.io/images/gc-sample-compact.png"/>
