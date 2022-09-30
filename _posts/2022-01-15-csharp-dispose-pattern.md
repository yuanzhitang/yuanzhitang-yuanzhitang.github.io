---
layout: post
title: "实现Dispose Pattern"
date: 2022-01-15 08:26:31
categories: .NET
tags: C# .NET Dispose
author: Michael
mathjax: true
---

* content
{:toc}

实现Dispose方法其实主要是用来释放托管资源。而一个类型中如果有嵌套有另一个实现IDisposable接口的实例成员时，通常会级联调用Dispose方法。当然一个类中如果只使用了托管资源，也是可以实现Dispose方法去尽早释放资源从而释放已经分配的内存，比如清空一个List，Dictionary等。

针对非托管资源，GC并不负责释放，那么有两个途径可以释放它们：
- 人为写代码主动去释放非托管资源
- 以防止人为忘记释放，给类添加终结器，类似于C++的析构函数（也不是完全相同，因为不知道而时会被调用)




### 类中没有非托管资源
针对类中没有非托管资源，那么类中不需要定义终结器。下面是推荐的模板代码：
类中定义私有变量记录是否已经disposed，调用Dispose()方法去释放资源后，将_disposed标记为true。
如果此时从他处再次调用Dispose(),因为_disposed已经为true，在Dispose(bool)方法中将直接返回。

```cs
class BaseClassWithoutFinalizer : IDisposable
{
	// To detect redundant calls
	private bool _disposed = false;

	// Public implementation of Dispose pattern callable by consumers.
	public void Dispose()
	{
		Dispose(true);
	}

	// Protected implementation of Dispose pattern.
	protected virtual void Dispose(bool disposing)
	{
		if (_disposed)
		{
			return;
		}

		if (disposing)
		{
			// TODO: dispose managed state (managed objects).
		}

		_disposed = true;
	}
}

```

### 释放非托管资源
例子中假定使用了非托管资源ptr, 下面是推荐的dispose模板代码：
首先实现IDisposable接口，同事定义一个protected virtual Dispose(bool)方法。同时定义~BaseClassWithFinalizer() 终结器。

在Dispose方法中调用Dispose(true), 这里的true表示确定性释放资源，也就是可以释放托管和非托管资源。资源释放后，使垃圾回收器不再需要调用对象的 Object.Finalize 方法。 因此，调用 SuppressFinalize 方法去阻止垃圾回收器运行终结器。 如果类型没有终结器，则对 GC.SuppressFinalize 的调用不起作用。 

在终结器中，调用Dispose(false)去释放非托管资源。这里主要是为了当人为忘记调用Dispose方法后，程序依然可以安全的释放非托管资源。
```cs
class BaseClassWithFinalizer : IDisposable
{
	// To detect redundant calls
	private bool _disposed = false;
	private IntPtr ptr = Marshal.AllocHGlobal(2);

	~BaseClassWithFinalizer() => Dispose(false);

	// Public implementation of Dispose pattern callable by consumers.
	public void Dispose()
	{
		Dispose(true);
		GC.SuppressFinalize(this);
	}

	// Protected implementation of Dispose pattern.
	protected virtual void Dispose(bool disposing)
	{
		if (_disposed)
		{
		return;
		}

		if (disposing)
		{
		// TODO: dispose managed state (managed objects).
		}

		// TODO: free unmanaged resources (unmanaged objects) and override a finalizer below.
		Marshal.FreeHGlobal(ptr);

		_disposed = true;
	}
}

```

这里的Dispose(bool disposing)采用virtual，主要是为了如果子类中如果有资源需要释放，那么可以重新此virtual方法。

下面是推荐的模板方法，一个关键点是disposing需要传递给父类的dispose(bool)方法。
```cs
class DerivedClassWithFinalizer : BaseClassWithFinalizer
{
    // To detect redundant calls
    private bool _disposedValue;

    ~DerivedClassWithFinalizer() => this.Dispose(false);

    // Protected implementation of Dispose pattern.
    protected override void Dispose(bool disposing)
    {
        if (!_disposedValue)
        {
            if (disposing)
            {
                // TODO: dispose managed state (managed objects).
            }

            // TODO: free unmanaged resources (unmanaged objects) and override a finalizer below.
            // TODO: set large fields to null.
            _disposedValue = true;
        }

        // Call the base class implementation.
        base.Dispose(disposing);
    }
}
```
