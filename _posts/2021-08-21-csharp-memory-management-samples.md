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
首先分配10给num，这个时候将num赋值给secNum时，其实是将值复制了一份给secNum，也就是说在stack中，有两个10。 如果此时将secNum修改为20，并不会影响到num。所有最后的输出是 ```10,20```

### Array
Array值存处在Heap上，在Stack上存储了执行Heap的地址信息。
看下面这个例子
<img width="100%" src="https://yuanzhitang.github.io/images/memory-alloc-array.png"/>
最后将会输出
```
1, 5
100, 5
100, 5
```
数组的传递其实是引用传递，所以当调用Change去修改数组的值时，Heap上面的将从2变成了5。
同理将第一个数组赋值给第二个数组，当修改了第二个数组的值时，也会影响到第一个数组的实际值，因为他们的值时同一份。