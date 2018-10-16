Here you'll learn how to create a new StyletIoC container, and register your services on it.

---
><font color="#63aebb" face="微软雅黑">在这里，你将学习如何创建一个新的StyletIoC容器，并在其上注册你的服务。</font>

Getting Started - The Builder - 入门 - 构建器
-----------------------------

Creating a new container is a two-step process. First you must create a new `StyletIoCBuilder`, and register all of your services on it. When you're done, you call `StyletIoCBuilder.BuildContainer()`, which will create a new `IContainer`. This ensures that a container is completely immutable, which simplifies quite a few things.

---
><font color="#63aebb" face="微软雅黑">创建新容器需要两个步骤。首先，你必须创建新的 `StyletIoCBuilder`，并在其上注册所有服务。完成后，调用 ` StyletIoCBuilder.BuildContainer()`，它将创建新的 `IContainer`。这确保了容器是完全不可变的，这简化了很多事情。

>示例：</font>
For example:

```csharp
// First, create the builder
// 首先，创建构建器
var builder = new StyletIoCBuilder();

// Configure the builder
// 配置构建器
builder.Bind<IVehicle>().To<HotHatchback>();

// When you're done, create a new `IContainer` from the builder
// 完成后，从构建器创建新的 `IContainer`
IContainer ioc = builder.BuildContainer();

// Now you can use this to resolve instances of services
// 现在你可以使用它来解析服务
var vehicle = ioc.Get<IVehicle>();
```

Different Ways Of Creating Types - 不同的创建类型方法
--------------------------------

As discussed previously, the purpose of StyletIoC is to take a request for a service type, and return an instance of something implementing that service, using the rules you configured using the `StyletIoCBuilder`. It can use a couple of different techniques to create that instance, though.

---
><font color="#63aebb" face="微软雅黑">如前所述，StyletIoC 的目的是接受对服务类型的请求，并使用 `StyletIoCBuilder` 配置的规则返回实现该服务的某个实例。不过，它可以使用一些不同的技术来创建实例。</font>
### Type Bindings - 类型绑定

The simplest is called a "type binding". You tell it the service type, and the type which implements that service, and it will figure out how to construct that type itself. For example:

---
><font color="#63aebb" face="微软雅黑">简单的称为“类型绑定”。告诉它服务类型和实现该服务的类型，它就知道如何构造该类型本身。例如：</font>

```csharp
builder.Bind<IVehicle>().To<HotHatchback>();
```

This tells StyletIoC "whenever I ask you for an `IVehicle`, create a new `HotHatchback` using an appropriate constructor, with all of the necessary dependencies passed in".

You can of course bind a service to itself, so long as that service is a concrete type. This is called a "self binding". For example:

---
><font color="#63aebb" face="微软雅黑">告诉 StyletIoC :“每当我向你请求 `IVehicle` 时，使用合适的构造函数创建新的 `HotHatchback`，并传递所有必要的依赖项。”

>当然，你可以将服务绑定到自身，只要服务是具体类型。这就是所谓的“自绑定”。例如：</font>

```csharp
builder.Bind<HotHatchback>().To<HotHatchback>();
// Or, more clearly:
builder.Bind<HotHatchback>().ToSelf();
```

This tells StyletIoC "whenever I ask you for a `HotHatchback`, give me a `HotHatchback` with all of its dependencies already populated".

You can also use the non-generic overloads if you wish:

---
><font color="#63aebb" face="微软雅黑">这告诉 StyletIoC：“每当我向你请求 `HotHatchback` 时，给我 `HotHatchback`，并填充所有依赖关系。”

>如果你愿意，也可以使用非泛型重载：</font>

```csharp
builder.Bind(typeof(IVehicle)).To(typeof(HotHatchback));
```

### Factory Bindings - Factory Bindings

If, for some reason, you want to tell StyletIoC *exactly* how to construct your type, you can pass it a delegate to call. This is called a "factory binding". For example:

---
><font color="#63aebb" face="微软雅黑">如果出于某种原因，你想告诉 StyletIoC 如何 *正确的* 构造你的类型，你可以将一个委托传递给它来调用。这被称为“工厂绑定”。例如:</font>

```csharp
builder.Bind<IVehicle>().ToFactory(container => new HotHatchback());
```

The `container` parameter there is an `IContainer`, and you can use it if your constructor needs dependencies injecting into it, for example:

---
><font color="#63aebb" face="微软雅黑">`container` 有一个 `IContainer` 参数，如果你的构造函数需要嵌入依赖项，你可以使用它，例如：</font>

```csharp
builder.Bind<IVehicle>().ToFactory(container => new HotHatchback(container.Get<Engine>()));
```

Of course, self-bindings work here too:

---
><font color="#63aebb" face="微软雅黑">当然，自绑定也适用于：</font>

```csharp
builder.Bind<HotHatchback>().ToFactory(container => new HotHatchback());
```

### Instance Bindings - 实例绑定

You can construct an instance of a type yourself and give it to the IoC container.
This can be useful for things like configuration objects.
For example:

---
><font color="#63aebb" face="微软雅黑">你可以自己构造一个类型的实例提供给IoC容器。这对配置对象这样的事情很有用。例如：</font>

```csharp
builder.Bind<IVehicle>().ToInstance(new HotHatchback());
```

Instance bindings are automatically singleton - the container doesn't know how to construct new instances of the type, and so has to always return the same instance.

By default the IoC container will dispose `IDisposable` instance types when it is disposed, but you can configure this using `.ToInstance(foo).DisposeWithContainer(false);`.

---
><font color="#63aebb" face="微软雅黑">实例绑定是自动单例的 - 容器不知道如何构造类型的新实例，因此必须总是返回相同的实例。

默认情况下，IoC 容器在处理 `IDisposable`实例类型时，会自动销毁，可以使用`.ToInstance(foo).DisposeWithContainer(false);` 来配置。</font>

Different Scopes - 不同的范围
----------------

### Transient Scope - 临时范围

By default, StyletIoC will create a new instance of a given type every time you ask it - this is called "transient scope". See:

---
><font color="#63aebb" face="微软雅黑">默认情况下，StyletIoC 会在每次请求时创建一个给定类型的新实例——这被称为“临时作用域”。如下:</font>

```csharp
var car1 = ioc.Get<IVehicle>();
var car2 = ioc.Get<IVehicle>();

// The following statement will succeed.
// 下面语句成功
Assert.AreNotEqual(car1, car2);
```

The IoC container won't dispose `IDisposable` transient types - it doesn't claim ownership over them, and doesn't know when they should be disposed.

---
><font color="#63aebb" face="微软雅黑">IoC 容器不会处理 `IDisposable` 临时类型 - 它不会声明对它们的所有权，也不知道什么时候应该处理。</font>

### Singleton Scope - 单例范围

However, you can also tell it to only create one instance of a service, ever, and return that one instance every time you ask it. This is called "singleton scope", and is very useful if your application does have singletons (most do), as it saves you from having a traditional Singleton object, which are notoriously hard to mock for unit tests.

---
><font color="#63aebb" face="微软雅黑">你也可以告诉它只创建一个服务实例，并在每次请求时返回该实例。这被称为“单例范围”，如果你的应用程序确实有单例（大多数都有），它非常有用，因为它可以避免使用传统的单例对象，这对于单元测试来说是非常难以模拟的。</font>

```csharp
builder.Bind<IConfigurationManager>().To<ConfigurationManager>().InSingletonScope();
var ioc = builder.BuildContainer();

var configManager1 = ioc.Get<IConfigurationManager>();
var configManager2 = ioc.Get<IConfigurationManager();

// The following statement will succeed.
// 下面语句成功
Assert.AreEqual(configManager1, configManager2);
```

Of course, this also works with factory bindings (where the factory will only be called once, ever):

---
><font color="#63aebb" face="微软雅黑">当然，这也适用于工厂绑定(此情况下，工厂只会被调用一次)：</font>

```csharp
builder.Bind<IConfigurationManager>().ToFactory(container => new ConfigurationManager()).InSingletonScope();
```

Note that singletons are scoped to the binding in which they're defined, see:

---
><font color="#63aebb" face="微软雅黑">注意，单例对象的作用域仅限于定义它们的绑定，如下：</font>

```csharp
builder.Bind<IConfigurationManager>().To<ConfigurationManager>().InSingletonScope();
builder.Bind<ConfigurationManager>().ToSelf();

var ioc = builder.BuildContainer();

var configManager1 = ioc.Get<IConfigurationManager>();
var configManager2 = ioc.Get<ConfigurationManager>();

// The following statement will succeed.
// 下面语句成功
Assert.AreNotEqual(configManager1, configManager2);
```

If you *really* want these both to return the same instance, you can use this technique:

---
><font color="#63aebb" face="微软雅黑">如果你真的希望这两者都返回相同的实例，可以使用这个技巧:</font>

```csharp
builder.Bind<ConfigurationManager>().And<IConfigurationManager>().To<ConfigurationManager>().InSingletonScope();
```

The IoC container will dispose `IDisposable` singletons when it is disposed.

---
><font color="#63aebb" face="微软雅黑">IoC 容器在处理 `IDisposable` 时会处理单例。</font>

Binding Multiple Types To A Single Service - 将多种类型绑定到单个服务
------------------------------------------

So far, we've experimented with binding a single type to a single service. That's not how it has to be, though. You *can* bind multiple types to a single service, for example:

---
><font color="#63aebb" face="微软雅黑">到目前为止，我们已经尝试将单个类型绑定到单个服务。但事实并非如此。你可以将多个类型绑定到一个服务，例如：</font>

```csharp
interface IVehicle { ... }
class HotHatchback : IVehicle { ... }
class OldBanger : IVehicle { ... }

builder.Bind<IVehicle>().To<HotHatchback>();
builder.Bind<IVehicle>().To<OldBanger>();

var ioc = builder.BuildContainer();

// This will throw an exception
// 这会抛出异常
ioc.Get<IVehicle>();

// This will return { new HotHatchback(), new OldBanger() }
// 将返回 { new HotHatchback(), new OldBanger() }
IEnumerable<IVehicle> vehicles = ioc.GetAll<IVehicle>();
```

As you can see, if you want to fetch an array of items, you need to use `IContainer.GetAll` - if you try a fetch a single `IVehicle` with `IContainer.Get`, StyletIoC doesn't know which to give you, so it throws an exception.

This also works with constructor and parameter injection, see:

---
><font color="#63aebb" face="微软雅黑">如你所见，如果你想获取 item 的数组，使用 `IContainer.GetAll` - 如果你尝试用 `IContainer.Get` 来获取一个 `IVehicle`，StyletIoC 不知道该给你哪个，所以它会抛出一个异常。

>这也适用于构造函数和参数注入，如下:
</font>

```csharp
class Garage
{
   public Garage(IEnumerable<IVehicle> vehicles) { ... }
}

// And

class Garage
{
   [Inject]
   public IEnumerable<IVehicle> Vehicles { get; set; }
}
```

Binding Generic Types - 绑定泛型类型
---------------------

StyletIoC handles bound generic types (where the exact types of the type parameters is known) the same as ordinary types, for example:

---
><font color="#63aebb" face="微软雅黑">StyletIoC 处理绑定泛型类型（其中已知类型参数的确切类型）与普通类型相同，例如：</font>

```csharp
interface IValidator<T> { ... }
class IntValidator : IValidator<int> { ... }
builder.Bind<IValidator<int>>().To<IntValidator>();
```

and

```csharp
interface IValidator<T> { ... }
class Validator<T> : IValidator<T> { ... }
builder.Bind<IValidator<int>>().To<Validator<int>>();
```

The fun begins when you want to create bindings for generic types where the exact types of the type parameters *aren't* known, for example:

---
><font color="#63aebb" face="微软雅黑">有趣的是，当你想为泛型类型创建绑定时，类型参数的确切类型是未知的，例如:</font>

```csharp
interface IValidator<T> { ... }
class Validator<T> : IValidator<T> { ... }
builder.Bind(typeof(IValidator<>)).To(typeof(Validator<>));

var ioc = builder.BuildContainer();

var intValidator = ioc.Get<IValidator<int>>(); // Returns a Validator<int>
```

Both the service and its implementation can have as many type parameters as you want, but they must have the *same* number of type parameters (it makes sense if you think it through). However, the type parameters can appear in any order:

---
><font color="#63aebb" face="微软雅黑">服务及其实现都可以包含任意数量的类型参数，但它们必须具有相同数量的类型参数（如果你仔细考虑，这是有意义的）。但是，类型参数可以按任何顺序出现：</font>

```csharp
interface ISomeInterface<T, U> { ... }
class SomeClass<U, T> : ISomeInterface<T, U> { ... }
builder.Bind(typeof(ISomeInterface<,>)).To(typeof(SomeClass<,>));
```

StyletIoC does not take type constraints into account - if you have a `interface IMyInterface<T> where T : class` and request an `IMyInterface<int>`, you'll get an exception.

---
><font color="#63aebb" face="微软雅黑">StyletIoC 不考虑类型约束 - 如果你有一个 `interface IMyInterface<T> where T : class` 并请求 `IMyInterface<int>`，你会得到一个异常。</font>

Autobinding - 自动绑定
-----------

StyletIoC is capable of creating bindings automatically for you.

---
><font color="#63aebb" face="微软雅黑">StyletIoC能够自动为你创建绑定。</font>

### Autobinding All Concrete Types - 自动绑定所有具体类型

To search the current assembly for all concrete types, and self-bind all of them, simple call

---
><font color="#63aebb" face="微软雅黑">要搜索当前程序集中的所有具体类型，并自行绑定所有这些类型，只需调用即可。</font>

```csharp
builder.Autobind();
```

There are overloads for passing in other assemblies to search, as well.

This is useful in an MVVM application, as it allows StyletIoC to resolve any of your ViewModels.

---
><font color="#63aebb" face="微软雅黑">同样，还有一些重载允许你指定要搜索的程序集。

这在 MVVM 应用程序中很有用，因为它允许 StyletIoC 解析任何 ViewModel。</font>

### Binding a service to all implementations - 将服务绑定到所有实现

You can also bind a service to all types which implement it, for example:

---
><font color="#63aebb" face="微软雅黑">你还可以将服务绑定到实现它的所有类型，例如：</font>

```csharp
interface IVehicle { ... }
class HotHatchback : IVehicle { ... }
class OldBanger : IVehicle { ... }

builder.Bind<IVehicle>().ToAllImplementations();

var ioc = builder.BuildContainer();

IEnumerable<IVehicle> vehicles = ioc.GetAll<IVehicle>(); 
// Returns { new HotHatchback(), new OldBanger() }
```

Again, there are also overloads allowing you to specify which assemblies to search.

This can be useful on its own (think finding all plugins), but is particularly useful when combined with unbound generics. For example:

---
><font color="#63aebb" face="微软雅黑">同样，还有一些重载允许你指定要搜索的程序集。

>这本身就很有用(想想查找所有插件)，在与未绑定类型的泛型结合使用时尤为有用。例如：</font>

```csharp
interface IValidator<T> { ... }
class IntValidator : IValidator<int> { ... }
class StringValidator : IValidator<string> { ... }

builder.Bind(typeof(IValidator<>)).ToAllImplementations();

var ioc = builder.BuildContainer();

var intValidator = ioc.Get<IValidator<int>>(); // Returns an IntValidator
var stringValidator = ioc.Get<IValidator<string>>(); // Returns a StringValidator
```

If you want more complex binding rules, StyletIoC doesn't provide an API for you - it barely takes any effort to do it yourself, and providing an API is just adding a lot of complexity for very little gain.

However, StyletIoC does define a couple of extension methods on Type which may make your life easier:

---
><font color="#63aebb" face="微软雅黑">如果你想要更复杂的绑定规则，StyletIoC 不会为你提供这个API - 你自己仅仅需要少许努力来做它，而提供一个API 只是增加了很多复杂性而获得的收益很少。

>然而，StyletIoC确实在Type上定义了一些扩展方法，这可能会让你更轻松：</font>

```csharp
// Returns all base types
someType.GetBaseTypes();

// Returns all base types and interfaces
someType.GetBaseTypesAndInterfaces();

// Returns true if someType implements someServiceType
someType.Implements(someServiceType)
// Also takes into account generics - so this is true:
typeof(Validator<int>>.Implements(typeof(IValidator<>));
```

[目录](./../Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[StyletIoC-Injection - StyletIoC 注入](./StyletIoC-Injection.md)