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
数组的传递其实是引用传递，所以当调用ChangeArray去修改数组的值时，Heap上面的将从2变成了5。
同理将第一个数组赋值给第二个数组，当修改了第二个数组的值时，也会影响到第一个数组的实际值，因为他们的值是同一份。

### List
List内部实现采用的是数组，初始容量是4.
比如：这个list初始化存储1，2，3，4
```cs
var list = new List<int> { 1, 2, 3, 4 };
```
如果此时调用list.Add方法添加5，6
```cs
list.Add(5);
list.Add(6);
```
因为list初始容量4已经被用完，这个时候List内部数组将会从堆上进行Resize，新的容量是8，从而可以放得下5和6
如果想避免进行Reszie，同时预支将会需要多大容量，可以在New List时就指定容量。
```cs
int expectedLength = 6;

var list = new List<int>(expectedLength) { 1, 2, 3, 4 };

list.Add(5);
list.Add(6);
```

### string
字符串是一种特别的引用类型，拥有同样的字符串值的变量其实是指向同一块Heap区域。
如果对字符串进行更改，那么新字符串值时重新分配的，不会更改原来字符串的值，也就是常说的immutable.
看下面这个代码
```
string str = "100";

string str2 = str;

Console.WriteLine(ReferenceEquals(str, str2));

str2 = "200";
Console.WriteLine(ReferenceEquals(str, str2));
Console.WriteLine($"{str},{str2}");


string strA = "A";
string strB = "B";
Console.WriteLine(ReferenceEquals(strA, strB));

strB = "A";
Console.WriteLine(ReferenceEquals(strA, strB));
```
将会依次输出：
```cs
true    //同一个引用
false   //str2被修改为200，是一份新地址
100,200 //
false   //字符串值不一样，不同地址
true    //字符串值变回A，又重新指向Heap上值为A的空间
```