The Problem - 问题
-----------

Constructor / parameter injection is all well and good, so long as you only need *one* instance of something. When you start needing to create instances yourself (think your ShellViewModel wanting to show dialogs, and needing to create ViewModels for them), i.e.

---
><font color="#63aebb" face="微软雅黑">构造函数/参数注入非常好，只需要一个实例即可。当你需要自己创建实例时(假设 ShellViewModel 想要显示对话框，并且需要为它们创建 ViewModel)。</font>

```csharp
class ShellViewModel
{
   // ...
   public void ShowDialog()
   {
      // We don't want to create DialogViewModel directly, as it has its own dependencies which need injecting

      //我们不想直接创建 DialogViewModel，因为它有需要注入的依赖项
      var dialogVm = new DialogViewModel();
      this.windowManager.ShowDialog(dialogVm);
   }
}
```

The tempting solution us to make your ShellViewModel (or whatever) aware of your IoC container, so it can call `this.container.Get<DialogViewModel>()`, like this:

---
><font color="#63aebb" face="微软雅黑">让你的 ShellViewModel (或其他任何东西)知道你的 IoC 容器是一个诱人的解决方案，因此它可以调用  `this.container.Get<DialogViewModel>()`，如下所示：</font>

```csharp
class ShellViewModel
{
   // ...
   public ShellViewModel(IContainer container)
   {
      this.container = container;
   }

   public void ShowDialog()
   {
      var dialogVm = this.container.Get<DialogViewModel>();
      this.windowManager.ShowDialog(dialogVm);
   }
}
```

This is known as the Service Locator Pattern, and there's a lot of debate as to whether it's actually an anti-pattern - essentially, it adds a dependency that shouldn't be added (your class needs to know about your (correctly configured) IoC container) - while hiding dependencies that your class actually does have.

---
><font color="#63aebb" face="微软雅黑">这被称为服务定位模式，关于它是否实际上是反模式存在很多争论 - 实质上，它增加了一个不应该添加的依赖项（你的类需要知道你的正确配置的IoC容器） - 同时隐藏你的类实际上具有的依赖性。</font>

The Solution - 解决方案
------------

This problem was actually solved in 1994 by the GoF - the Abstract Factory Pattern. Since you can't inject a `DialogViewModel` into your `ShellViewModel`, you can instead inject a factory which can create `DialogViewModels`, like this:

---
><font color="#63aebb" face="微软雅黑">这个问题实际上是在1994年由GoF(抽象工厂模式)解决的。因为不能注入一个 `DialogViewModel` 到 `ShellViewModel`，你可以注入一个工厂创建 `DialogViewModels`，像这样：</font>

```csharp
class ShellViewModel
{
   // ...
   public ShellViewModel(Func<DialogViewModel> dialogViewModelFactory, IWindowManager windowManager)
   {
      this.dialogViewModelFactory = dialogViewModelFactory;
      this.windowManager = windowManager;
   }

   public void ShowDialog()
   {
      var dialogVm = this.dialogViewModelFactory();
      this.windowManager.ShowDialog(dialogVm);
   }
}
```

This works really well: you can provide whatever you like here for testing, and the IoC container can provide something suitable for normal operation.

StyletIoC supports two sorts of factories: "Func-factories" - like the one shown above - and "abstract factories" - where you provide an interface for the factory and StyletIoC provides the implementation. Both are documented in their own sections below.

---
><font color="#63aebb" face="微软雅黑">这非常有效：你可以提供你喜欢的任何测试，IoC 容器可以提供适合正常操作的东西。

>StyletIoC 支持两种类型的工厂:`Func-factories`(如上面所示)和 `abstract factories`(即为工厂提供接口，StyletIoC提供实现)。这两种方法都在下面各自的部分中进行了记录。</font>

Func Factories - 功能工厂
--------------

These are the easiest types of factories to use. They require no setup on your part, and they're quite flexible. They do however have a couple of limitations: these are discussed later, and are solved by abstract factories.

The golden rule is this: If you can execute `container.Get<ISomeInterface>()`, you can also execute `container.Get<Func<ISomeInterface>()`. The same goes of course for constructor and property injection: if you can write a constructor like this:

---
><font color="#63aebb" face="微软雅黑">这是最容易使用的工厂类型。它们不需要你进行任何设置，而且非常灵活。然而，它们有一些限制：这些将在后面讨论，并由抽象工厂解决。

>黄金法则是这样的：如果你可以执行 `container.Get<ISomeInterface>()`，你也可以执行 `container.Get<Func<ISomeInterface>()`。构造函数和属性注入当然也是如此：你可以这样编写构造函数：</font>

```csharp
class Foo
{
   public Foo(IBar bar)
   {
   }
}
```

---
><font color="#63aebb" face="微软雅黑">你也可以这样写：</font>

```csharp
class Foo
{
   public Foo(Func<IBar> bar)
   {
   }
}
```

If you execute a func factory `Func<ISomeInterface>` multiple times, the effect is the same as if you'd called `container.Get<ISomeInterface>()` multiple times: if `ISomeInterface` was registered as a singleton, you'll get the same instance back each time. If it was registered as a transient, you'll get a different instance back each time.

Func factories also have some other tricks up their sleeves: If you request a `Func<IEnumerable<ISomeInterface>>`, (either using `container.Get<Func<IEnumerable<ISomeInterface>>>()`, or through constructor/property injection), you'll get a factory that, when called, will return the same thing as `container.GetAll<ISomeInterface>()` (or if you'd injected `IEnumerable<ISomeInterface>` into your constructor/property). 

Similarly, if you request an `IEnumerable<Func<ISomeInterface>>` (either through `container.GetAll<Func<ISomeInterface>()`, or by injecting `IEnumerable<Func<ISomeInterface>>` into a constructor or property) you'll get a collection of factories, and each one will be able to create a new instance of that implementation of `ISomeInterface`.

Func factories do have a limitation: fetching stuff that requires a key. Recall that you can call `container.Get<ISomeInterface>("magicKey")`? While you can call `var factory = container.Get<Func<ISomeInterface>>("magicKey"); factory()`, you *can't* do something like `var factory = container.Get<Func<string, ISomeInterface>>(); factory("magicKey")`. This is a deliberate decision - it's all a bit non-obvious to start with, and there are some really tricky corner-cases when you start getting involved with collections of factories and factories of collections. If you need this behaviour, use an Abstract Factory (below).

---
><font color="#63aebb" face="微软雅黑">如果你多次执行功能工厂 `Func<ISomeInterface>`，效果就像你多次调用 `container.Get<ISomeInterface>()`:如果 `ISomeInterface` 注册为单例，你每次都会得到相同的实例。如果它注册为临时，每次都会返回一个不同的实例。

>功能工厂还有一些其他的技巧：如果你请求 `Func<IEnumerable<ISomeInterface>>`（使用 `container.Get<Func<IEnumerable<ISomeInterface>>>()` 或通过构造函数/属性注入），你将得到一个工厂，当被调用时，将返回与 `container.GetAll<ISomeInterface>())` 相同的值(或者如果你将 `IEnumerable<ISomeInterface>` 注入你的构造函数/属性)。

>类似，如果你请求 `IEnumerable<Func<ISomeInterface>>` （通过 `container.GetAll<Func<ISomeInterface>>()`，或通过注入 `IEnumerable<Func<ISomeInterface>>` 构造函数或属性），你将获得一个工厂集合，并且每个工厂都将能够创建该实现的新实例 `ISomeInterface`。

>功能工厂确实有一个限制：获取需要 Key 内容。回想一下，你可以调用 `container.Get<ISomeInterface>("magicKey")`？你可以调用 `var factory = container.Get<Func<ISomeInterface>>("magicKey"); factory()`，但你不能像这样 `var factory = container.Get<Func<string, ISomeInterface>>(); factory("magicKey")`。这是一个深思熟虑的决定 - 从一开始就不明确，当你开始涉及工厂和工厂集合时，有一些非常棘手的角落情况。如果你需要此行为，请使用抽象工厂（如下所示）。</font>


Abstract Factories - 抽象工厂
-----------------

Abstract factories are similar to func factories, but instead of every factory having the type `Func<ISomeInterface>`, you instead define your own factory.

Consider this example:

```csharp
interface IDialogViewModelFactory
{
   DialogViewModel CreateDialogViewModel();
}

class ShellViewModel
{
   // ...
   public ShellViewModel(IDialogViewModelFactory dialogViewModelFactory)
   {
      this.dialogViewModelFactory = dialogViewModelFactory;
   }

   public void ShowDialog()
   {
      var dialogVm = this.dialogViewModelFactory.CreateDialogViewModel();
      this.windowManager.ShowDialog(dialogVm);
   }
}
```

In the most basic case, you can provide a simple implementation of `IDialogViewModelFactory` which just calls into the IoC container for production use, or you can provide a mock for testing.

But writing that implementation of `IDialogViewModelFactory` is a bit of a pain, and one which StyletIoC has a solution for. Like a few other IoC containers, StyletIoC is capable of taking an interface like this, and generating an implementation for you (the difference being that StyletIoC does not require any dependencies to do this).

To take advantage of this, use the binding syntax `builder.Bind<IFactoryInterfaceType>().ToAbstractFactory()`, for example:

```csharp
public interface IDialogViewModelFactory
{
   DialogViewModel CreateDialogViewModel();
}

// ... 

builder.Bind<IDialogViewModelFactory>().ToAbstractFactory();
```

Such factory methods can also create collections of types, for example:

```csharp
public interface IVehicleFactory
{
   IEnumerable<IVehicle> CreateVehicleCollection();
}
```

### Restrictions


There are a few things to bear in mind when writing your factory interface. These are expanded on below:

1. The interface must be public, or you must have `[assembly: InternalsVisibleTo(StyletIoC.FactoryAssemblyName)]` in your `AssemblyInfo.cs`.
2. Every method in the factory must return a type which is registered with the IoC container, or an IEnumerable of that type, and must have either zero parameters, or a single string parameter.

StyletIoC generates an implementation of the factory interface in a different assembly, so your interface must either be public, or you must have marked this assembly as being a 'friend' of yours (using `[assembly: InternalsVisibleTo(StyletIoC.FactoryAssemblyName)]`). You probably do want your factory interfaces to be public (so you can manually provide an implementation when writing your unit tests.

StyletIoC only knows how to generate method implementations for methods which return something (i.e. that aren't `void`), and which take zero parameters, or a single string parameter (used as a key, covered below).

### Using Keys With Abstract Factories

There are a number of different ways you specify the key to use when StyletIoC's implementation creates a new instance of a type (see [[StyletIoC Keys]]). The first is using the `[Inject(Key = "key")]` attribute:

```csharp
public interface IDialogViewModelFactory
{
   [Inject(Key = "someKey")]
   DialogViewModel CreateDialogViewModel();
}
```

You can also create methods which take a single string parameter, which will be used:

```csharp
public interface IDialogViewModelFactory
{
   DialogViewModel CreateDialogViewModel(string key);
}

// ... then

this.dialogViewModel.CreateDialogViewModel("someKey");
```

### Under The Hood

Under the hood, StyletIoC generates a type with an implementation which looks a lot like this:

```csharp
public interface IFactoryInterface
{
   TypeWithoutKey CreateTypeWithoutKey();

   IEnumerable<ElementType> CreateCollection();

   [Inject(Key = "key")]
   TypeWithAttributeKey CreateTypeWithAttributeKey();

   TypeWithKeyParameter CreateTypeWithKeyParameter(string key);
}

public class FactoryInterface : IFactoryInterface
{
   private IContainer container;
   public FactoryInterface(IContainer container)
   {
      this.container = container;
   }

   public TypeWithoutKey CreateTypeWithoutKey()
   {
      return this.container.GetTypeOrAll(typeof(TypeWithoutKey));
   }

   public IEnumerable<ElementType> CreateCollection()
   {
      return this.container.GetTypeOrAll(typeof(IEnumerable<ElementType>));
   }

   public TypeWithAttributeKey CreateTypeWithAttributeKey()
   {
      return this.container.GetTypeOrAll(typeof(TypeWithAttributeKey), "key");
   }

   public TypeWithKeyParameter CreateTypeWithKeyParameter(string key)
   {
      return this.container.GetTypeOrAll(typeof(TypeWithKeyParameter), key);
   }
}
```

[目录](./../Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[StyletIoC-Modules - StyletIoc 模块](./StyletIoC-Modules.md)