---
layout: post
title:  "C# BigInteger 处理超大整型数字"
date:   2018-01-19 22:46:17
categories: .NET
tags: C#
author: Michael
mathjax: true
---

* content
{:toc}

今天遇到一个要处理XSD中Integer的数值区间的计算的问题，Integer这个类型的值区间理论上是可没有边界的，假设目前的值是1.5E+10000, 这个数字已经达到double和Int64都无法存储了，同时我还要对如此大的数字进行加减运算，后来发现了BigInteger这个类可以很好的解决我遇到的问题。^_^



### BigInteger
自.net framework 4.0开始引入， 位于命名空间：
```cs
namespace System.Numerics
```
设计用于存储超大整型数字，所以只要内存够大，存储是没有上限和下限的，否则如果数字过大的话，会遇到OutOfMemory的异常。

### 我的案例
因为我的输入就是一个字符串的数字，所以我调用BigInteger.Parse()方法可以得到一个BigInteger实例，然后就可以对于进行+1 或者 -1的运算了
```cs
static void Main(string[] args)
        {

            String largeNum = "1000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000";

            var number = BigInteger.Parse(largeNum);
            var numberDecreaseOne = number - 1;
            var numberIncreaseOne = number + 1;

            Console.WriteLine(numberDecreaseOne);
            Console.WriteLine(" ");
            Console.WriteLine(numberIncreaseOne);
            Console.ReadKey();

        }
```
#### 输出结果
<img width="90%" src="https://yuanzhitang.github.io/images/Csharp-output-big-integer.png"/>


C# BigInteger 处理超大整型数字

BigInteger还很很多的方法：比如 **Min, Max, Substract, Multiply, Divide, Log, Pow**, 等等，同时BigInteger对大量的运算符都进行了重载，很方便使用。

更多资料可以参看MSDN [System.Numerics.BigInteger](https://docs.microsoft.com/en-us/dotnet/api/system.numerics.biginteger?redirectedfrom=MSDN&view=net-5.0)