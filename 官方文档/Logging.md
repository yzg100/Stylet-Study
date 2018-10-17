Introduction
------------

Sometimes it's useful to see what Stylet's doing under the hood, particularly if it's not doing something you expect it to, or doing something unexpected.

Thankfully, Stylet can be easily configured to produce logging output, so you can get an insight into what it's doing.

Quick Start
-----------

To quickly enable logging, put the following in your Bootstrapper's Configure method:

```csharp
protected override void OnStart()
{
   Stylet.Logging.LogManager.Enabled = true;
}
```

This will print log messages to the Output window in Visual Studio. Internally, the default logger uses `Trace.WriteLine`.


Customising the Logging
-----------------------

You can of course provide your own logger to Stylet, which Stylet will use to print log messages.

First, define a class which implements the `Stylet.Logging.ILogger` interface:

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

```csharp
protected override void OnStart()
{
   Stylet.Logging.LogManager.LoggerFactory = name => new MyLogger(name);
   Stylet.Logging.LogManager.Enabled = true;
}
```

Logging within your Application
-------------------------------

I'd advise against using Stylet.Logging elsewhere in your logging. It's very lightweight and has almost no features - the only reason to write it was so that Stylet didn't have dependencies on logging frameworks to support a feature which will almost never be used.

[NLog](http://nlog-project.org/) and [log4net](http://logging.apache.org/log4net/) are the two major C# logging frameworks. If you don't want to couple your application to any particular logging framework, consider [Common.Logging](http://netcommon.sourceforge.net/), which provides a framework-agnostic interface, behind which NLog, log4net, or another framework can be wired in.