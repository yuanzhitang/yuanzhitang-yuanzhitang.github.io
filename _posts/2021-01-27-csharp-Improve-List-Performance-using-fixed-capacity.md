---
layout: post
title: "C# List指定初始容量以提升性能"
date: 2021-01-27 20:15:26
categories: .NET
tags: C# List Performance Benchmark.NET
author: Michael
mathjax: true
---

- content
  {:toc}

C# List 类型默认的初始容量是 4，如果随着元素的增加，List 内存是采用翻倍的方式去增加系统的容量，这样则会导致内存需要进行 Resizing，那么如果可以预期到最大的容量需求，如何避免 Resizing 带来的性能损耗呢？

本文主要涵一下几个方面：

- 演示 List 自动扩容
- List 手动缩容
- 传递初始容量给 List
- 比较默认和指定初始容量（Initial Capacity)的性能差异



## List 自动扩容

C# `List` 是一个范型集合，可以存储大量的对象在内存中，默认创建一个 `List` 对象并不会分配空间，一旦添加第一个元素时，List 第一次将会分配空间为 4 的容量，如果持续添加元素之集合中，则会分配 8，接着是 16，这样成倍的增加。这个过程叫做 Resizing。`List` 的属性 `Capacity` 可以看到当前 `List` 对象的空间。

```cs
static void Main(string[] args)
{
	var list = new List<string>();

	Console.WriteLine($"Initial Capacity: {list.Capacity}");

	for (int i = 0; i < 10; i++)
	{
		list.Add(i.ToString());

		Console.WriteLine($"List Size: {list.Count}, Capacity: {list.Capacity}");
	}
}
```

这段代码输入如下：

```shell
Initial Capacity: 0
List Size: 1, Capacity: 4
List Size: 2, Capacity: 4
List Size: 3, Capacity: 4
List Size: 4, Capacity: 4
List Size: 5, Capacity: 8
List Size: 6, Capacity: 8
List Size: 7, Capacity: 8
List Size: 8, Capacity: 8
List Size: 9, Capacity: 16
List Size: 10, Capacity: 16
```

## List 手动缩容

List `TrimExcess`可以一次 Trim 掉多余的空间。

```cs
static void Main(string[] args)
{
	var list = new List<string> { "1", "2", "3", "4", "5", "6", "7", "8", "9", "10" };
	Console.WriteLine($"Current Capacity: {list.Capacity}");
	for (int i = 1; i <= 10; i++)
	{
		list.Remove(i.ToString());

		list.TrimExcess();

		Console.WriteLine($"List Size: {list.Count}, Capacity: {list.Capacity}");
	}
}
```

这段代码输入如下，可以看到当移除玩所有的元素了，最后 List 其实是保留了一个元素的 Capacity。

```shell
Current Capacity: 16
List Size: 9, Capacity: 9
List Size: 8, Capacity: 9
List Size: 7, Capacity: 7
List Size: 6, Capacity: 7
List Size: 5, Capacity: 5
List Size: 4, Capacity: 5
List Size: 3, Capacity: 3
List Size: 2, Capacity: 3
List Size: 1, Capacity: 1
List Size: 0, Capacity: 1
```

## 传递初始容量给 List

为了避免前面看到 Resizing 的发生，注意如果是我们已经提前知道需要存储多少元素就可以直接指定 List 的 Capacity。
可以通过构造函数指定。
比如需要存放 26 个英文字母到下面的 list 中，则可以直接：
var list = new List<string>(26).

## 比较默认和指定初始容量（Initial Capacity)的性能差异

频繁的进行 Resizing 将会对性能造成一定的损耗，下面对比了往 List 中添加不同数量的元素时，指定 Capacity 和不指定的性能差异。性能的对比是采用了 Benchmark.NET.

六个测试方法分别是：

- FewElementsList 写入少量元素到默认 List
- FewElementsFixedList 写入少量元素到指定 Capacity 的 List
- NormalElementsList 写入常规数量元素到默认 List
- NormalElementsFixedList 写入常规数量元素到指定 Capacity 的 List
- LargeElementsList 写入大量元素到默认 List
- LargeElementsFixedList 写入大量元素到指定 Capacity 的 List

ListTest.cs

```cs
using BenchmarkDotNet.Attributes;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ListPerformanceTest
{
	public class ListTest
	{
		[Benchmark]
		public void FewElementsList()
		{
			var oldList = new List<string>() { "a", "b", "c"};

			var newList = new List<string>();

			foreach(var item in oldList)
			{
				newList.Add(item);
			}
		}

		[Benchmark]
		public void FewElementsFixedList()
		{
			var oldList = new List<string>() { "a", "b", "c"};

			var newList = new List<string>(oldList.Count);

			foreach (var item in oldList)
			{
				newList.Add(item);
			}
		}

		[Benchmark]
		public void NormalElementsList()
		{
			var oldList = new List<string>() { "a", "b", "c", "d", "e", "f"};

			var newList = new List<string>();

			foreach (var item in oldList)
			{
				newList.Add(item);
			}
		}

		[Benchmark]
		public void NormalElementsFixedList()
		{
			var oldList = new List<string>() { "a", "b", "c", "d","e", "f"};

			var newList = new List<string>(oldList.Count);

			foreach (var item in oldList)
			{
				newList.Add(item);
			}
		}

		[Benchmark]
		public void LargeElementsList()
		{
			var oldList = new List<string>() { "a", "b", "c", "d","f","g","h","i","j","k","l" };

			var newList = new List<string>();

			foreach (var item in oldList)
			{
				newList.Add(item);
			}
		}

		[Benchmark]
		public void LargeElementsFixedList()
		{
			var oldList = new List<string>() { "a", "b", "c", "d", "f", "g", "h", "i", "j", "k", "l" };

			var newList = new List<string>(oldList.Count);

			foreach (var item in oldList)
			{
				newList.Add(item);
			}
		}
	}
}

```

Program.cs 调用 BenchmarkRunner.Run<ListTest>();去对比测量前面的6个方法

```cs
using BenchmarkDotNet.Running;
using System;

namespace ListPerformanceTest
{
	class Program
	{
		static void Main(string[] args)
		{
			BenchmarkRunner.Run<ListTest>();

			Console.ReadLine();
		}
	}
}

```

在 Release 模式下面运行测试程序，将会得到如下测试结果（不用机器测试结果数字可能不同，主要看性能差异）：
从Mean列可以看到当元素较少(<4)时，指定Capacity并没有性能优势，但是当要添加的元素超过4之后，大约有15%左右的性能提升。

```
|                  Method |      Mean |     Error |    StdDev |
|------------------------ |----------:|----------:|----------:|
|         FewElementsList |  84.09 ns |  1.618 ns |  1.662 ns |
|    FewElementsFixedList |  94.29 ns |  2.742 ns |  7.956 ns |
|      NormalElementsList | 199.38 ns |  5.799 ns | 17.098 ns |
| NormalElementsFixedList | 166.27 ns |  4.813 ns | 13.887 ns |
|       LargeElementsList | 354.74 ns | 13.096 ns | 38.613 ns |
|  LargeElementsFixedList | 304.44 ns | 11.457 ns | 32.503 ns |

// * Warnings *
MultimodalDistribution
  ListTest.NormalElementsList: Default      -> It seems that the distribution can have several modes (mValue = 3.12)
  ListTest.NormalElementsFixedList: Default -> It seems that the distribution can have several modes (mValue = 2.96)
  ListTest.LargeElementsList: Default       -> It seems that the distribution is bimodal (mValue = 3.61)
Environment
  Summary -> Benchmark was executed with attached debugger

// * Hints *
Outliers
  ListTest.FewElementsList: Default         -> 2 outliers were removed (92.74 ns, 93.24 ns)
  ListTest.FewElementsFixedList: Default    -> 3 outliers were removed (118.43 ns..123.85 ns)
  ListTest.NormalElementsFixedList: Default -> 4 outliers were removed (211.80 ns..263.95 ns)
  ListTest.LargeElementsFixedList: Default  -> 7 outliers were removed (411.32 ns..498.23 ns)

// * Legends *
  Mean   : Arithmetic mean of all measurements
  Error  : Half of 99.9% confidence interval
  StdDev : Standard deviation of all measurements
  1 ns   : 1 Nanosecond (0.000000001 sec)
```

## 结论

- `List<T>`的容量是 `List<T>`可以容纳的元素数量。当元素被添加到 `List<T>`时，容量会根据需要通过重新分配内部数组自动增加。
- 如果可以估计集合的大小，那么指定初始容量就不需要在向 `List<T>`添加元素时执行大量调整大小的操作。
- 可以通过调用 `TrimExcess` 方法或显式设置 `capacity` 属性来减少容量。减少容量重新分配内存并复制 `List<T>`中的所有元素。

参考资料：
[MSDN List<T>(Int32)](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.-ctor?view=net-5.0#System_Collections_Generic_List_1__ctor_System_Int32_)
[BenchmarkDotNet](https://benchmarkdotnet.org/)
