Introduction - 介绍
------------

Sometimes it's useful to see what Stylet's doing under the hood, particularly if it's not doing something you expect it to, or doing something unexpected.

Thankfully, Stylet can be easily configured to produce logging output, so you can get an insight into what it's doing.

---
><font color="#63aebb" face="微软雅黑">有时看看 Stylet 在后台做什么是有用的，特别是如果它没有做你期望的事情，或做一些意想不到的事情。

值得庆幸的是，Stylet 可以轻松配置生成日志记录输出，因此你可以深入了解它正在做什么。</font>

Quick Start - 快速入门
-----------

To quickly enable logging, put the following in your Bootstrapper's Configure method:

---
><font color="#63aebb" face="微软雅黑">要快速启用日志记录，请将以下内容放入 Bootstrapper 的 Configure 方法中：</font>

```csharp
protected override void Configure()
{
   Stylet.Logging.LogManager.Enabled = true;
}
```

This will print log messages to the Output window in Visual Studio. Internally, the default logger uses `Trace.WriteLine`.

---
><font color="#63aebb" face="微软雅黑">这将把日志消息打印到 Visual Studio 的输出窗口。在内部，默认日志记录器使用 `Trace.WriteLine`。</font>


Customising the Logging - 自定义日志记录
-----------------------

You can of course provide your own logger to Stylet, which Stylet will use to print log messages.

First, define a class which implements the `Stylet.Logging.ILogger` interface:

---
><font color="#63aebb" face="微软雅黑">当然，你可以为 Stylet 提供自己的日志记录器，Stylet 将使用它来打印日志消息。

>首先，定义一个实现 `Stylet.Logging.ILogger` 接口的类：</font>

```csharp
public class MyLogger : Stylet.Logging.ILogger
{
   public MyLogger(string loggerName)
   {
      // TODO
   }

   public void Info(string format, params object[] args)
   {
      // TODO
   }

   public void Warn(string format, params object[] args)
   {
      // TODO
   }

   public void Error(Exception exception, string message = null)
   {
      // TODO
   }
}
```

Then, configure the LogManager to use it. As before, in your Bootstrapper's Configure method:

然后，配置 LogManager 使用它。一样在 Bootstrapper 的 Configure 方法中配置：

```csharp
protected override void Configure()
{
   Stylet.Logging.LogManager.LoggerFactory = name => new MyLogger(name);
   Stylet.Logging.LogManager.Enabled = true;
}
```

Logging within your Application - 在应用程序中记录日志
-------------------------------

I'd advise against using Stylet.Logging elsewhere in your logging. It's very lightweight and has almost no features - the only reason to write it was so that Stylet didn't have dependencies on logging frameworks to support a feature which will almost never be used.

我建议不要在其他地方使用 Stylet.Logging 记录日志。它非常轻量级，几乎没有任何功能 - 编写它的唯一原因是，Stylet 不需要依赖日志框架来支持一个几乎永远不会被使用的特性。

[NLog](http://nlog-project.org/) and [log4net](http://logging.apache.org/log4net/) are the two major C# logging frameworks. If you don't want to couple your application to any particular logging framework, consider [Common.Logging](http://netcommon.sourceforge.net/), which provides a framework-agnostic interface, behind which NLog, log4net, or another framework can be wired in.

[NLog](http://nlog-project.org/) 和 [log4net](http://logging.apache.org/log4net/) 是两个主要的C＃ 日志框架。如果你不想将应用程序耦合到任何特定的日志记录框架，请考虑使用 [Common.Logging](http://netcommon.sourceforge.net/)，它提供了一个框架无关的接口，在该接口后面可以连接NLog，log4net或其他框架。

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[LabelledValue](./LabelledValue.md)