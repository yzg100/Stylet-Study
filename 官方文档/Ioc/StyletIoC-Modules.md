Any application with a reasonable amount of complexity will include a lot of IoC container configuration: there are lots of explicit binding of interfaces to implementations, of specifying singletons, and other (completely necessary) configuraiton.

The default approach is to put all of this configuration in your bootloader, and this makes sense: there's one single place where all of the configuration happens, and it's easy to see what's going on.

However, as your design grows, you might find yourself looking for a way to split up the IoC container configuration a bit. This is the role of modules.

I think an example is the best way of demonstrating how these work. I'm not going to include hundreds of lines of IoC configuration just for the sake of an example - I'll only include a few, and let you imagine the rest.


---
><font color="#63aebb" face="微软雅黑">任何具有合理复杂性的应用程序都将包含大量IoC容器配置：接口与实现，指定单例和其他（有必要的）配置有很多显式绑定。

>默认方法是将所有这些配置放在引导加载程序中，这是有道理的：只有一个地方可以完成所有配置，并且很容易看到发生。

>但是，随着设计的增长，你可能会发现自己正在寻找一种方法来拆分 IoC 容器配置。这就是模块的作用。

>我认为一个例子是展示这些如何工作的最佳方式。我不会仅仅为了一个例子而包含数百行 IoC 配置 - 我只会包含一些，让你想象其余部分。</font>

Before - 之前:

```csharp
class Bootstrapper : Bootstrapper<IShellViewModel>
{
   protected override void ConfigureIoC(IStyletIoCBuilder builder)
   {
      builder.Bind<IServiceAComponent1>().To<ServiceAComponent1>().InSingletonScope();
      builder.Bind<IServiceAComponent2>().To<ServiceAComponent2>().InSingletonScope();
      builder.Bind<ISomeFactory>().ToAbstractFactory();
   }
}
```

After - 之后:

```csharp
class ServiceAModule : StyletIoCModule
{
   protected override void Load()
   {
      Bind<IServiceAComponent1>().To<ServiceAComponent1>().InSingletonScope();
      Bind<IServiceAComponent2>().To<ServiceAComponent2>().InSingletonScope();
   }
}

class FactoriesModule : StyletIoCModule
{
   protected override void Load()
   {
      Bind<ISomeFactory>().ToAbstractFactory();
   }
}

class Bootstrapper : Bootstrapper<IShellViewModel>
{
   protected override void ConfigureIoC(IStyletIoCBuilder builder)
   {
      builder.AddModule(new ServiceAModule());
      builder.AddModule(new FactoriesModulie());
   }
}
```

As you can see, we've been able to break up the container configuration into multiple modules. Each module is responsible for doing a bit of container configuration, and then the bootstrapper adds all of these modules to the container. While there is a bit more code, you've got a tool to separate and modularise things a bit, which is sometimes very useful.

Modules are simply classes which inherit from the abstract `StyletIoCModule`. There is one abstract method you must override `void Load()`: this is called when the module is added to the container, and is responsible for adding all of your bindings. If you try and add bindings anywhere else (for example in the constructor), they won't have any effect.

There are two protected methods available on `StyeltIoCModule` for you to use: `Bind<TService>()` and `Bind(Type serviceType)`. These work exactly the same as the methods on `IStyletIoCBuilder`, and you use these to create your bindings.

Finally, there's a method on `IStyletIoCBuilder` called `AddModule`, which takes a module instance as an argument. This simply runs the module's `Load` method, and adds all of the resulting bindings to itself. If you add the same module twice, you'll get duplicating bindings - simple.

---
><font color="#63aebb" face="微软雅黑">如你所见，我们已经能够将容器配置分解为多个模块。每个模块负责进行一些容器配置，然后 bootstrapper 将所有这些模块添加到容器中。虽然有很多的代码，但你有一个工具来分离和模块化一些内容，这非常有用。

>模块只是继承自 `StyletIoCModule` 抽象类。你必须重写抽象方法 `void Load()`：将模块添加到容器时调用此方法，并负责添加所有绑定。如果你尝试在其他地方添加绑定（例如在构造函数中），它们将不会产生任何影响。

>`StyeltIoCModule` 中有两个受保护的方法可供你使用:`Bind<TService>()` 和 `Bind(Type serviceType)`。它们的工作原理与 `IStyletIoCBuilder` 中的方法完全相同，你可以使用它们来创建绑定。

>最后，在 `IStyletIoCBuilder` 中有一个 `AddModule` 方法，它接受一个模块实例作为参数。这只是简单地运行模块的 `Load` 方法，并将所有结果绑定添加到自身。如果添加两次相同的模块，你将得到重复的绑定 — 非常简单。</font>

[目录](./../Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[StyletIoC-Misc - StyletIoC 杂项](./StyletIoC-Misc.md)