---
layout: post
title: "C#中如何高效的拼接字符串？"
date: 2021-11-24 09:11:24
categories: .NET
tags: C# .NET String
author: Michael
mathjax: true
---

* content
{:toc}

字符串拼接是一个非常常见的需求，目前C#中也是支持多种方式进行字符串拼接。

常见的方式有：
- 用+号
- string.concat
- StringBuilder
- $"{string}"
- string.format
- string.join

那么到底应该采用何种方式最佳呢？




### 拼接两个字符串
如果只是对两个字符串进行拼接，比如将"Hello" 和 "World" 拼接成 "HelloWorld"，下面采用BenchMark.NET对这些方式进行对比，看看性能差异如何.
```cs
[MemoryDiagnoser]
public class ConcatTwoStringsBenchmarks
{
	private string Hello = "Hello";
	private string World = "World";

	[Benchmark]
	public string Orginal()
	{
		return Hello + World;
	}

	[Benchmark]
	public string Concat()
	{
		return string.Concat(Hello, World);
	}

	[Benchmark]
	public string StringBuilder()
	{
		var stringBuilder = new StringBuilder();
		stringBuilder.Append(Hello);
		stringBuilder.Append(World);
		return stringBuilder.ToString();
	}

	[Benchmark]
	public string StringBuilderFixedLength()
	{
		var stringBuilder = new StringBuilder(Hello.Length+World.Length);

		stringBuilder.Append(Hello);
		stringBuilder.Append(World);
		return stringBuilder.ToString();
	}

	[Benchmark]
	public string Dollar()
	{
		return $"{Hello}{World}";
	}

	[Benchmark]
	public string StringFormat()
	{
		return string.Format("{0}{1}", Hello, World);
	}

	[Benchmark]
	public string Join()
	{
		return string.Join("", new string[] { Hello, World });
	}
}

static void Main(string[] args)
{
    BenchmarkRunner.Run<ConcatTwoStringsBenchmarks>();
}

```
运行结果如下：从图中可以看到采用 +, string.concat 和 $ 差异不大，采用StringBuilder反而更慢。
<img width="100%" src="https://yuanzhitang.github.io/images/compare-concat-two-strings.png"/>

### 拼接多个字符串
依然采用Benchmark.NET，这次让程序依次拼接2, 5，50和100个字符串，看看性能差异如何.
```cs
[MemoryDiagnoser]
public class ConcatMultiStringsBenchmarks
{
	private string[] _myArray;
	
	[Params(2, 5, 50, 100)]
	public int Size { get; set; }
	
	public int Capacity { get; set; }
	
	[GlobalSetup]
	public void Setup()
	{
		_myArray = new string[Size];
		Capacity = 0;
		for (var i = 0; i < Size; i++)
		{
		var str = i.ToString();
		_myArray[i] = str;
		Capacity += str.Length;
		}
	}
	[Benchmark(Baseline = true)]
	public string Original()
	{
		var str = string.Empty;
		for (int i = 0; i < _myArray.Length; i++)
		{
			str += _myArray[i];
		}
		return str;
	}

	[Benchmark]
	public string Concat()
	{
		var str = string.Empty;
		for (int i = 0; i < _myArray.Length; i++)
		{
			str = string.Concat(str, _myArray[i]);
		}
		return str;
	}
	[Benchmark]
	public string StringBuilder()
	{
		var stringBuilder = new StringBuilder();
		for (int i = 0; i < _myArray.Length; i++)
		{
			stringBuilder.Append(_myArray[i]);
		}
		return stringBuilder.ToString();
	}
	[Benchmark]
	public string StringBuilderFixedLength()
	{
		var stringBuilder = new StringBuilder(Capacity);
	…
	}
	[Benchmark]
	public string Join()
	{
		return string.Join("", _myArray);
	}
}

static void Main(string[] args)
{
    BenchmarkRunner.Run<ConcatMultiStringsBenchmarks>();
}

```
运行结果如下：
<img width="100%" src="https://yuanzhitang.github.io/images/compare-concat-multiple-strings.png"/>

结论：
- 如果对小数量的字符串进行拼接， +, string.concat性能更好。
- 对大量字符串拼接，如果可以预知多少容量，那么使用StringBuilder并指定初始Size。 如果无法预测容量，那么就采用StringBuilder
