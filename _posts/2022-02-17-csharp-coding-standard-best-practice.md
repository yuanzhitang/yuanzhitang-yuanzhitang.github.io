---
layout: post
title: "C# Coding Standard 最佳实践"
date: 2022-02-17 11:45:41
categories: .NET
tags: C# .NET Coding
author: Michael
mathjax: true
---

* content
{:toc}

团队中，代码规范是一件很重要的约定，每一个人都应该去遵守，这样才可以写出大家都容易读懂的代码。本文列出了常用的代码标准规范与约定。
主要涵盖：
- 类名，变量名
- 常量，缩写
- 下划线，类型名
- 接口，文件名
- 命名空间
- 括号，枚举




### 类名
推荐：采用PascalCasing命名方式给类名和方法名
```cs
public class ClientActivity
{
    public void ClearStatistics()
    {
        //...
    }
    public void CalculateStatistics()
    {
        //...
    }
}

```
理由：和.NET命名一致，易读
