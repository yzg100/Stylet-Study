StyletIoC is a very lightweight and extremely fast IoC container. It was designed to do only a few things, but to do them very well and in an intuitive manner.

It's configured using a fluent interface - none of this XML rubbish. It also has zero dependencies.

I'm going to assume for now that you're reasonably confident with the concept of an IoC container - if not, do some reading and come back. I may write a more in-depth introduction in the future.

---
><font color="#63aebb" face="微软雅黑">StyletIoC 是一个非常轻量级和非常快的 IoC 容器。它只用于做一些事情，但要以直观的方式做好。 

>它使用流畅的接口配置 - 没有XML，也没有依赖关系。

>我现在要假设你对 IoC 容器的概念有相当的信心 - 如果没有，去看看再回来。我以后可能会写一篇更深入的介绍。</font>

Services and Implementations - 服务和实现
----------------------------

StyletIoC is constructed around the concept of a service. A service is a concrete type, abstract type, or interface, which is (or can be) implemented by a concrete type, for example:

---
><font color="#63aebb" face="微软雅黑">StyletIoC 围绕服务的概念构建。服务是具体类，抽象类或接口，它是（或可以）由具体的类实现，例如：</font>

```csharp
interface IVehicle { ... }
class HotHatchback : IVehicle { ... }
```

Here, `IVehicle` is the service, and `HotHatchback` is the concrete type which implements it. Note that `HotHatchback` is also a service - one that is implemented by the `HotHatchback` class itself.

When you configure StyletIoC, you define a set of relationships. Each relationship is between a service, and the type (or types) which implement it. So here, we could tell StyletIoC "create a relationship between the service `IVehicle`, and the type `HotHatchback`".

Later, when you want an implementation of `IVehicle`, you can ask StyletIoC "give me an instance of something which implements the service `IVehicle`, and StyletIoC wil construct a `HotHatchback` and pass it back to you.

---
><font color="#63aebb" face="微软雅黑">`IVehicle` 是服务，`HotHatchback` 是实现它的具体类。请注意，`HotHatchback` 也是一个服务 - 由 `HotHatchback` 类本身实现的服务。

>配置 StyletIoC 时，可以定义一组关系。每个关系都在服务和实现它的类型（或类型）之间。所以在这里，我们可以告诉 StyletIoC 在服务 `IVehicle` 和类型 `HotHatchback` 之间建立关系。

>当你想要实现 `IVehicle`时， 向 StyletIoC 请求一个实现该服务的实例 `IVehicle`，而 StyletIoC 将构建一个 `HotHatchback` 并将其传给你。</font>

Resolving Types - The Service Locator and Injection - 解析类型 - 服务定位器和注入
---------------------------------------------------

There are 3 ways to get StyletIoC to construct a type for us:

1. By calling `IContainer.Get` directly
2. Constructor Injection
3. Property Injection

Calling `IContainer.Get` directly is the easiest to explain, and looks something like this:

---
><font color="#63aebb" face="微软雅黑">有三种方法可以让 StyletIoC 为我们构建一个类型：
>1. 直接调用 `IContainer.Get`
>2. 构造函数注入
>3. 属性注入

>直接调用 `IContainer.Get` 是最容易解释的，如下：</font>

```csharp
var ioc = ... // Covered in lots of detail elsewhere
var vehicle = ioc.Get<IVehicle>();
```

As tempted as this may look, this should only be done in the root of your application - use Constructor Injection and Parameter Injection elsewhere.

When StyletIoC constructs a type for you, it will look for a constructor which has parameters of types which it knows how to resolve. It will then resolve those types, and inject them into the constructor. For example:

---
><font color="#63aebb" face="微软雅黑">尽管这看起来很诱人，但这只应该在应用程序的根目录中完成——在其他地方使用构造函数注入和参数注入。

>当 StyletIoC 为你构造一个类型时，它将查找一个构造函数，该构造函数具有它知道如何解析的类型参数。然后它将解析这些类型，并将它们注入构造函数中。例如：</font>

```csharp
class Engine { ... }
class Car
{
   public Car(Engine engine)
   {
      // 'engine' contains a new instance of Engine
   }
}
```

This is the most common way of new instances of things out of StyletIoC - every type which StyletIoC constructs lists its dependencies in its constructor, and StyletIoC will construct each of those, injecting *its* dependencies as it does.

If you wish, you can also do parameter injection, provided that the parameter to be injected has the attribute `[Inject]`, for example:

---
><font color="#63aebb" face="微软雅黑">这是 StyletIoC 创建新实例的最常见方式 - StyletIoC 构造的每一种类型都在它的构造函数中列出它的依赖关系并注入依赖关系。

>如果你愿意，你也可以使用参数注入，前提是要注入的参数具有 `[Inject]` 属性，例如：</font>

```csharp
class Engine { ... }
class Car
{
   [Inject]
   public Engine Engine { get; set; }
}
```

The various merits of each are discussed elsewhere.

---
><font color="#63aebb" face="微软雅黑">其他优点在别处讨论。</font>

[目录](./../Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[StyletIoC-Configuration - StyletIoC 配置](./StyletIoC-Configuration.md)