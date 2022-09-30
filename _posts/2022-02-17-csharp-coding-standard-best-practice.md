---
layout: post
title: "C# Coding Standard 最佳实践"
date: 2022-02-17 11:45:41
categories: .NET
tags: C# .NET 代码规范
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
推荐：采用PascalCasing命名方式给类名和方法名, 同时类型名词或者名字短语组合。
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

public class Employee
{
}
public class BusinessLocation
{
}
public class DocumentCollection
{
}

```
理由：和.NET命名一致，易读。也很容易记住。

### 变量名
推荐：针对局部变量和方法参数采用camelCasing命名.
```cs
public class UserLog
{
    public void Add(LogEvent logEvent)
    {
        int itemCount = logEvent.Items.Count;
        // ...
    }
}
```
理由：和.NET命名一致，易读

### 标识符
不推荐：不要使用带有匈牙利标识符或者其他类型的前缀标识符。
```cs
// Correct
int counter;
string name;
 
// Avoid
int iCounter;
string strName;
```
理由：和.NET命名一致，易读。除此之外，Visual Studio中用Tooltips可以很容易知道一个变量的类型。最好不要加这样的类型缩写作为前缀。

### 常量
推荐：采用PascalCasing命名方式给常量，不要采用全部大写的方式
```cs
// Correct
public static const string ShippingType = "DropShip";
 
// Avoid
public static const string SHIPPINGTYPE = "DropShip";
```
理由：和.NET命名一致，易读。全部大写容易引起过多的注意力，也不容易分辨出其中的单词组合。

### 缩写
推荐：一般情况下避免使用缩写，除非，缩写名是一个非常常用的名字，比如 Id, Xml, Ftp, Uri
```cs
// Correct
UserGroup userGroup;
Assignment employeeAssignment;
 
// Avoid
UserGroup usrGrp;
Assignment empAssignment;
 
// Exceptions
CustomerId customerId;
XmlDocument xmlDocument;
FtpHelper ftpHelper;
UriPart uriPart;
```
理由：和.NET命名一致，易读。同时可以组织不同的程序员写出不一致的缩写。

如果需要使用缩写，缩写名是3个或者以上的字符，采用PascalCasing，如果是两个字符，则全部大写。
像下面的： Html, Ftp, UI
```cs
HtmlHelper htmlHelper;
FtpTransfer ftpTransfer;
UIControl uiControl;
```
理由：和.NET命名一致，易读。大写视觉上可以吸引更好的注意。

### 下划线
推荐：不推荐在变量和方法中使用下划线，除非变量为局部变量
```cs
// Correct
public DateTime clientAppointment;
public TimeSpan timeLeft;
 
// Avoid
public DateTime client_Appointment;
public TimeSpan time_Left;
 
// Exception
private DateTime _registrationDate;
```
理由：和.NET命名一致，易读。读这个变量更加自然，不需要去重音读下划线。

### 类型名
推荐：使用预定义的类型名而不是系统的真实类型名，比如：Int16, Single, UInt64等。
```cs
// Correct
string firstName;
int lastIndex;
bool isSaved;
 
// Avoid
String firstName;
Int32 lastIndex;
Boolean isSaved;
```
理由：和.NET命名一致，读起来更加自然。

### 隐含类型(var)
推荐：使用var针对局部变量，除非类型是基本的简单类型（`int, string double`等）。
```cs
var stream = File.Create(path);
var customers = new Dictionary();
 
// Exceptions
int index = 100;
string timeSheet;
bool isCompleted;
```
理由：消除混乱，特别是复杂的泛型类型。类型很容易通过Visual Studio工具提示检测。。