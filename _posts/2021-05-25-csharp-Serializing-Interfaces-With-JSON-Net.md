---
layout: post
title: "采用JSON.NET序列化接口属性"
date: 2021-05-25 06:27:19
categories: .NET
tags: C# JSON.NET
author: Michael
mathjax: true
---

* content
{:toc}

当你的对象中的一个属性（`Property`）是一个接口，这个接口有几个实现的具体类时，如果采用 JSON.NET 进行简单快速的序列化和反序列化呢?

比如这个例子，一个 `Message`, 他的 `Header` 是一个接口，同时 `Message` 自己也是一个抽象类，它有两个子类分别是 `HttpMessage` 和 `RpcMessage`.



```cs
public class Message
{
	public IMessageHeader Header { get; set; }

	public object Body { get; set; }

	public Message()
	{

	}
}

public class HttpMessage : Message
{

}

public class RpcMessage : Message
{

}
```

`Header` 又分为 `HttpHeader` 和 `RpcHeader`：

```cs
public interface IMessageHeader
{
	string Name { get; set; }
}

public class HttpHeader : IMessageHeader
{
	public string Name { get; set; } = "Http";
}

public class RpcHeader : IMessageHeader
{
	public string Name { get; set; } = "Rpc";
}
```

下面我们来创建一些 `Message`。

```cs
	var messageList = new List<Message>
	{
		new HttpMessage()
		{
			Header = new HttpHeader(),
			Body = new byte[] { 1, 2, 3 }
		},
		new RpcMessage()
		{
			Header = new RpcHeader(),
			Body = new byte[] { 4, 5, 6 }
		}
	};
```

采用默认的对象去序列化这个 `messageList`。

```cs
	var jsonString = JsonConvert.SerializeObject(messageList);
```

产生的 JSON 数据将会如下。

```json
[
  { "Header": { "Name": "Http" }, "Body": "AQID" },
  { "Header": { "Name": "Rpc" }, "Body": "BAUG" }
]
```

但是当我们将 Json 反序列化为原始对象时

```cs
	var messages = JsonConvert.DeserializeObject<List<Message>>(jsonString);
```

将会遇到如下错误

```
Newtonsoft.Json.JsonSerializationException: 'Could not create an instance of type JSON_DEMO.IMessageHeader. Type is an interface or abstract class and cannot be instantiated. Path '[0].Header.Name', line 1, position 19.'
```

主要原因是 JSON.NET 本身并不知道它应该发序列化为哪一个具体的子类，为了解决这个问题，如果我们可以在序列化对象时，将子类的信息同时序列化到 Json 数据中，那么反序列化回对象时，就可以知道具体应该反序列化成哪一个具体的子类。

JSON.NET 提供了一个解决方案，在序列化和反序列化时，指定如下的设置。

```cs
	JsonSerializerSettings settings = new JsonSerializerSettings
	{
		TypeNameHandling = TypeNameHandling.Auto
	};
```

当序列化对象时，指定这个 `settings`

```cs
var jsonString = JsonConvert.SerializeObject(messageList, settings);
```

现在我们将会得到如下的 Json

```json
[
  {
    "$type": "JSON_DEMO.HttpMessage, JSON_DEMO",
    "Header": { "$type": "JSON_DEMO.HttpHeader, JSON_DEMO", "Name": "Http" },
    "Body": { "$type": "System.Byte[], mscorlib", "$value": "AQID" }
  },
  {
    "$type": "JSON_DEMO.RpcMessage, JSON_DEMO",
    "Header": { "$type": "JSON_DEMO.RpcHeader, JSON_DEMO", "Name": "Rpc" },
    "Body": { "$type": "System.Byte[], mscorlib", "$value": "BAUG" }
  }
]
```

当我们反序列化这个 Json 成 `List<Message>`是，将可以得到原始的对象。

```cs
var messages = JsonConvert.DeserializeObject<List<Message>>(jsonString, settings);
```

## 完整代码

```cs
using Newtonsoft.Json;
using System.Collections.Generic;

namespace JSON_DEMO
{
	class Program
	{
		static void Main(string[] args)
		{
			var messageList = new List<Message>
			{
				new HttpMessage()
				{
					Header = new HttpHeader(),
					Body = new byte[] { 1, 2, 3 }
				},
				new RpcMessage()
				{
					Header = new RpcHeader(),
					Body = new byte[] { 4, 5, 6 }
				}
			};

			JsonSerializerSettings settings = new JsonSerializerSettings
			{
				TypeNameHandling = TypeNameHandling.Auto
			};

			var jsonString = JsonConvert.SerializeObject(messageList, settings);
			var messages = JsonConvert.DeserializeObject<List<Message>>(jsonString, settings);
		}
	}

	public class Message
	{
		public IMessageHeader Header { get; set; }

		public object Body { get; set; }

		public Message()
		{

		}
	}

	public class HttpMessage : Message
	{

	}

	public class RpcMessage : Message
	{

	}

	public interface IMessageHeader
	{
		string Name { get; set; }
	}

	public class HttpHeader : IMessageHeader
	{
		public string Name { get; set; } = "Http";
	}

	public class RpcHeader : IMessageHeader
	{
		public string Name { get; set; } = "Rpc";
	}
}

```
