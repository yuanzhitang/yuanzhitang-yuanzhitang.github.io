---
layout: post
title: ".NET 内存管理"
date: 2022-08-10 10:25:19
categories: .NET
tags: C# .NET GC
author: Michael
mathjax: true
---

* content
{:toc}

.NET CLR 管理了应用程序占用内存的分配和释放，开发人员在开发托管的应用程序是就不用在开发代码去管理内了。针对内存的管理依赖于垃圾回收机制（Garbage Collection)，分为 标记(`Full Mark`) -> 回收(`Sweep`) -> 压缩(`Compact`）三个阶段，不在使用引用计数的方式了，类似采用GC机制的还有像Java,Node JS, Go 等编程语言。

<img width="100%" src="https://yuanzhitang.github.io/images/MemoryUsedByAProcess.png"/>
一个进程占用的内存主要分为Allocated Space、Free、Committed和Reserved。而在.NET应用程序中，在应用程序内转成原生的机器码之前，将由C#编译器先将原始代码和相关的资源与引用编译成托管应用程序集，之后交由CLR进行管理和相关的内存管理。
<img width="100%" src="https://yuanzhitang.github.io/images/managed-code-clr.png"/>

### Stack 和 Heap
.NET主要使用了堆栈、非托管堆和托管堆
- 堆栈(Stack)
  它是按每个线程管理的，用于存储局部变量、方法参数和临时值。当方法返回时，GC不会自动清理堆栈。对对象的引用存储在堆栈上，但实际对象在堆上分配，GC知道这一点。当GC无法找到对象的引用时，它将从堆中删除该对象。
  <img width="100%" src="https://yuanzhitang.github.io/images/stack.png"/>
- 非托管堆
  非托管代码将在非托管堆或堆栈上分配对象。托管代码还可以通过调用Win32 api在非托管堆上分配对象。
- 托管堆(Managed Heap)
  托管代码在托管堆上分配对象，而GC负责管理托管堆, 总共有三个代(Generation)，分别为Gen0, Gen1和Gen2。GC还维护一个大对象堆，以补偿在内存中移动大对象的成本。
  <img width="100%" src="https://yuanzhitang.github.io/images/heap.png"/>
  备注:
  最初，Gen0为256KB, Gen1为2MB, Gen2为10MB
  用于>=85000字节的对象的大对象堆(LOH)，可以采用` GC.GetGeneration (obj)`去获取当前对象存活于哪一个Gen。
  比如下面的代码，将会输出 0 0 2 0
```cs
public class LargeObjectExample
{
	public string NormalString { get; set; } = "Example";
	public byte[] ByteArray85000 = new byte[85000];
	public byte[] ByteArray80000 = new byte[80000];
}


static void Main(string[] args)
{
	var largeObj = new LargeObjectExample();

	Console.WriteLine(GC.GetGeneration(largeObj));
	Console.WriteLine(GC.GetGeneration(largeObj.NormalString));
	Console.WriteLine(GC.GetGeneration(largeObj.ByteArray85000));
	Console.WriteLine(GC.GetGeneration(largeObj.ByteArray80000));

	Console.ReadKey();
}
```


