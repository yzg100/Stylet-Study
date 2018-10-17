This page is background reading for someone wanting to dig into the StyletIoC source code, either to verify it or to submit a pull request.

---
><font color="#63aebb" face="微软雅黑">本页是为了让一些想要深入了解 StyletIoC 源代码的，或者是为了验证它，或者是为了提交 pull request 的人阅读而准备的。</font>

Expression: How To Quickly Instantiate a Type - 表达式：如何快速实例化类型
---------------------------------------------

The traditional way of creating an instance of a type at runtime is using `Activator.CreateIntance`, e.g. `Activator.CreateInstance(instanceType, new object[] { container.Get(param1Type), container.Get(param2Type) })`. However, this is deadly slow, and most IoC containers no longer use it (if they ever did).

在运行时创建类型实例的传统方法是使用 `Activator.CreateIntance`，例如 `Activator.CreateInstance(instanceType, new object[] { container.Get(param1Type), container.Get(param2Type) })`。然而，这是非常缓慢的，大多数 IoC 容器不再使用它。

C# 3 introduced Expression Trees, which were expended on by C# 4. Expression trees are many things, but one of their abilities is to allow you to generate an expression specifying some operation (e.g. `a + b`), and compile that down to IL at run-time, as a delegate. This allows you to write an expression using e.g. `Expression.New(intanceType)`, and compile that down to a delegate which is as fast as doing `new Instance()` yourself.

Many IoC containers have adopted this approach, and use it to write expressions which describe the code `new Instance(container.Get(param1Type), container.Get(param2Type))`. This provides a significant speed-up over `Activator.CreateInstance`, but still requires hitting the IoC container for each parameter to be resolved, which incurs additional overhead.

StyletIoC takes this one step further, and generates an expression which describes the code `new Instance(new Param1Instance(), new Param2Instance())`, which is significantly faster.

This trick is also used when one of the arguments requires property injection, where an Expression equivalent to the following is created:

---
><font color="#63aebb" face="微软雅黑">C# 3 引入了表达式树，C# 4 使用了表达式树。表达式树有很多东西，但是它们的功能之一是允许你生成一个表达式来指定一些操作（如：a + b），并在运行时将其编译为IL。这允许写一个表达式如：`Expression.New(intanceType)`，然后把它编译成一个委托，它的速度和你自己处理 `new Instance()` 一样快。

>很多 IoC 容器都采用了这种方法，并使用它来编写描述代码的表达式 `new Instance(container.Get(param1Type), container.Get(param2Type))`。这大大加快了 `Activator.CreateInstance` 处理速度，但仍然需要通过调用 IoC 容器来解析每个参数，这会带来额外的开销。

>StyletIoC 更进一步，生成一个描述代码的表达式，`new Instance(new Param1Instance(), new Param2Instance())` 速度明显更快。

>当其中一个参数需要属性注入时，也会使用此技巧，相当于创建以下表达式：</font>

```csharp
var param1 = new Param1Instance();
param1.SomeProperty = new SomePropertyInstance();
new Instance(param1, new Param2Instance());
```

Creators and Registrations - 创建与注册
--------------------------

StyletIoC is mainly centered around two important interfaces: `ICreator` and `IRegistration`.

An `ICreator` knows how to provide an instance of a type - the `TypeCreator` knows how to create an instance of a type (and is used if you register a type using `Bind<..>().To<...>()`), while the `FactoryCreator` knows how to create an instance using a factory you've specified (using `Bind<..>().ToFactory(...)`).

An `IRegistration` is responsible for the lifetime of an instance, and will create a new instance of the type where necessary using the `ICreator` which it owns. The `TransientRegistration` will create a new instance each time one's requested, while the `SingletonRegistration` will call its `ICreator` only once, and will cache the resulting instance. The `TransientRegistration` is used most of the time, but if you specify a singleton using `.InSingletonScope()`, a `SingletonRegistration` will be used.

There's a further piece to the puzzle, which is `IRegistrationCollection`, implemented by `SingleRegistration` (which owns a single `IRegistration`) and `RegistrationCollection` (which owns a collection of `IRegistration`s). This is mainly an optimisation - an `IRegistrationCollection` can be asked to supply either a single `IRegistration`, or multiple ones, and a `RegistrationCollection` will throw an exception in the former case.

At the heart of StyletIoC is a dictionary of `[serviceType, key] -> IRegistrationCollection`. When you call `IContainer.Get`, StyletIoC will find the correct `IRegistrationCollection` for the type you specified, and ask it for a single `IRegistration`. It will then ask that `IRegistration` to provide an instance of its type.

---
><font color="#63aebb" face="微软雅黑">StyletIoC 主要围绕两个重要的接口：`ICreator` 和 `IRegistration`。

>`ICreator` 知道如何提供一个类型的实例 - `TypeCreator` 知道如何创建一个类型的实例（如果你使用 `Bind<..>().To<...>()` 注册类型），而 `FactoryCreator` 知道如何使用你指定的工厂来创建实例（使用 `Bind<..>().ToFactory(...)`）。

>`IRegistration` 负责实例的生命周期，`ICreator` 根据其拥有的内容在必要时创建该类型的新实例。`TransientRegistration` 会在每个请求时创建一个新的实例，而 `SingletonRegistration` 只会调用 `ICreator`一次，并且会缓存结果实例。在使用 `TransientRegistration` 大多数情况下，如果你使用 `.InSingletonScope()` 指定一个单例对象，那么将使用 `SingletonRegistration`。

>这个困惑是 `IRegistrationCollection`，它由 `SingleRegistration`（拥有 `IRegistration`）和 `RegistrationCollection`（拥有 `RegistrationCollection`）实现。这主要是一种优化 - `IRegistrationCollection` 可以要求提供单个 `IRegistration` 或多个，但 `RegistrationCollection` 在前一种情况下将抛出异常。

>StyletIoC 的核心是 `[serviceType, key] -> IRegistrationCollection`。当你调用 `IContainer.Get` 时，StyletIoC 会为你找到正确类型的 `IRegistrationCollection`，向它请求一个 `IRegistration`。然后，它将要求 `IRegistration` 提供其类型的实例。</font>

'GetAll' Registrations - 'GetAll' 注册
----------------------

Things get slightly more complex when collections of registrations are requested. StyletIoC has a further dictionary of `[elementType, key] => IRegistration`, where that `IRegistration` is a `GetAllRegistration`. Given an `IEnumerable<T>`, you can extract the `T`, then use it to get an `IRegistration` from this dictionary. That `IRegistration` can create a `List<T>`, populated with all the correct elements.

This dictionary is consulted by `IContainer.GetAll`, and parts of the code responsible for constructor and property injection, when they encounter an `IEnumerable<T>`.

This dictionary is populated on-the-fly when when required.

---
><font color="#63aebb" face="微软雅黑">当请求注册的集合时，事情会变得稍微复杂一些。StyletIoC 有 `[elementType, key] => IRegistration` 字典，其中 ` IRegistration` 是 `GetAllRegistration`。给定 `IEnumerable<T>`，你可以提取 `T`，然后它从字典获取 `IRegistration `。`IRegistration` 可以创建一个 `List<T>`，并填充所有正确的元素。

>当遇到 `IEnumerable<T>` 时，`IContainer.GetAll` 以及负责构造函数和属性注入的代码将搜索字典。

>当需要时，字典是动态填充的。</font>

Unbound Generics - 无约束的泛型
----------------

When you register an unbound generic type (e.g. `builder.Bind(typeof(IValidator<>)).To(typeof(Validator<>))`), StyletIoC adds an entry to a dictionary of `[unboundGenericType, key] => List<UnboundGeneric>`. If you request a generic type which isn't in the registrations dictionary, StyletIoC will see whether it can construct that type using any of the entries in this dictionary. If it can, it will create a new `IRegistration`, and add it to the registrations dictionary.

---
><font color="#63aebb" face="微软雅黑">当你注册未指定的泛型类型（如：`builder.Bind(typeof(IValidator<>)).To(typeof(Validator<>))`）时，StyletIoC 会在字典中添加一条信息 `[unboundGenericType, key] => List<UnboundGeneric>`。如果你请求的泛型类型不在注册字典中，StyletIoC 将查看它是否可以使用此字典中的任何信息构造该类型。如果可以，它将创建新的 `IRegistration`，并将其添加到注册字典中。</font>

BuilderUppers
-------------

There's a final dictionary which completes the puzzle, one of `type => BuilderUpper`. Each BuilderUpper knows how to perform property injection on an instance of that type. When you call `IContainer.BuildUp` this is consulted, and the relevant `BuilderUpper` retrieved (or created if it didn't already exist) and used to build up your type. It's also used by the `ICreator`s to perform property injection.

---
><font color="#63aebb" face="微软雅黑">最后字典来结束 `type => BuilderUpper` 困惑 。每个 `BuilderUpper` 都知道如何在该类型的实例上执行属性注入。当你调用 `IContainer.BuildUp` 时，会检索 `BuilderUpper` 相关信息（如果信息尚未存在则创建）并用于构建你的类型。也用于 `ICreator` 执行属性注入。</font>

[目录](./../Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[The ViewManager - 视图管理](./../The-ViewManager.md)