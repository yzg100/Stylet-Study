This page goes into more detail on Constructor Injection and Property Injection.

---
><font color="#63aebb" face="微软雅黑">本页详细介绍了构造函数注入和属性注入。</font>

Constructor Injection - 构造函数注入
---------------------

Constructor injection happens automatically when StyletIoC creates a new instance of a type, provided that the type was bound using the `Bind<...>().To<...>()` syntax, and not `Bind<...>().ToFactory(...)`.

When presented with a type to construct, StyletIoC first needs to determine which constructor to call, if there's more than one:

1. If a constructor is decorated with the `[Inject]` attribute, StyletIoC will use that.
2. Any constructors which contain parameters with types which StyletIoC doesn't know how to resolve, and which don't have a default value, are discarded.
3. The constructor with the most parameters which StyletIoC can provide values for is selected, and default values used for parameters which StyletIoC can't provide values for.

If a constructor type is an `IEnumerable<T>` for some `T`, and you haven't explicitly registered that `IEnumerable<T>` as a service with StyletIoC, but you *did* register one or more types which implement `T`, StyletIoC will construct an `IEnumerable<T>` containing instances of those types.

---
><font color="#63aebb" face="微软雅黑">当 StyletIoC 创建类型的新实例时，构造函数注入会自动发生，前提是该类型是使用 `Bind<...>().To<...>()` 语法绑定的，而不是 `Bind<...>().ToFactory(...)`。

>当提出要构造的类型时，如果有多个构造函数 StyletIoC 首先需要确定要调用哪个构造函数：

>1. 如果构造函数被修饰为 `[Inject]`，那么 StyletIoC 会使用它。

>2. 任何包含 StyletIoC 不知道如何解析的类型参数的构造函数，以及没有默认值的构造函数，都会被忽略。

>3. StyletIoC 可以提供参数最多的构造函数被选中，而使用默认值的参数是 StyletIoC 无法提供值的。

如果构造类型是 `IEnumerable<T>` 的 `T`，并且你没有显式地将 `IEnumerable<T>` 注册为使用StyletIoC 的服务，但是你已经注册了一个或多个实现 `T` 的类型，StyletIoC 将构建一个 `IEnumerable<T>`，它包含这些类型的实例。</font>


Property Injection - 属性注入
------------------

After constructing a type, StyletIoC will inject values into all properties and fields decorated with the `[Inject]` attribute. It will inject into both public, protected and private properties and fields - it's not recommended to inject values into private members (since it becomes impossible to set up the type without reflection), but you may need it.

The main problem with property injection is that you properties should really be world-writable (so it can be set up without reflection, for example in a unit test), even if they shouldn't be from an encapsulation point of view. This often ends up with properties which are `{ set; private get; }`, which is really horrible. For this reason, constructor injection is almost always preferred.

Because the properties are injected after the type is constructed, everything which happens in the constructor must not rely on these properties being populated. This is another reason to prefer constructor injection. If you *do* need to know when the properties are injected, implement the interface `IInjectionAware`. The interface method `PropertiesInjected` will be called when all properties have been injected.

Because property injection requires properties to be decorared with `[Inject]`, your type can't be IoC-container-agnostic, as they can with constructor injection. This will bite you if you want to change IoC container.

As with constructor injection, a property of type `IEnumerable<T>` for some `T` will have a collection of that type injected.

You can also perform property injection if StyletIoC did not construct the type - call `IContainer.BuildUp`. For example:

---
><font color="#63aebb" face="微软雅黑">构造类型后，StyletIoC 会将值注入到使用 `[Inject]` 属性修饰的所有属性和字段中。它将注入公共属性，受保护属性和私有属性和字段 - 不建议将值注入私有成员（因为没有反射就无法设置类型），但你可能会需要它。

>属性注入的主要问题是你的属性应该是完全可写的（所以它可以在没有反射的情况下设置，例如在单元测试中），即使它们不应该从封装的角度来看。这往往会导致属性 `{ set; private get; }` ，非常可怕。因此，构造函数注入几乎总是首选的。

>因为在构造类型之后注入属性，所以构造函数中发生的所有内容都不能依赖于填充的这些属性。这是首选构造函数注入的另一个原因。如果确实需要知道何时注入属性，请实现该接口 `IInjectionAware`。`PropertiesInjected` 注入所有属性时将调用接口方法。

>由于属性注入需要用 `[Inject]` 来修饰属性，因此你的类型不能与 IoC 容器无关，因为它们可以使用构造函数注入。如果你想更改 IoC 容器，这将对你造成影响。

>与构造函数注入一样，`IEnumerable<T>` 某些类型的属性 `T` 将具有注入的该类型的集合。

>如果 StyletIoC 没有构造类型 - 调用 `IContainer.BuildUp`，你还可以执行属性注入，例如：</font>

```csharp
var car = new OldBanger():
ioc.BuildUp(car);
```

Examples:

```csharp
class OldBanger
{
   // Will not be injected - no [Inject] attribute
   // 不会被注入-没有 [Inject] 修饰属性
   public IEngine Engine { get; set; }
}

class OldBanger
{
   // Will be injected, provided a type is registered for the service IEngine
   // 将被注入，提供 IEngine 服务
   [Inject]
   public IEngine Engine { get; set; }
}

class Garage
{
   // Will inject all types registered for the service IVehicle
   // 将注入为服务中注册的所有 IVehicle 类型
   [Inject]
   public IEnumerable<IVehicle> Vehicles { get; set; }
}

class OldBanger
{
   // Fields are also injected into
   // 字段也被注入
   [Inject]
   public IEngine Engine;
}

class OldBanger
{
   // This works true, although you really shouldn't do it
   // 这也能正常工作，但你不应该这样做
   [Inject]
   private IEngine Engine;
}

class OldBanger : IInjectionAware
{
   [Inject]
   public IEngine Engine { get; set; }

   public PropertiesInjected()
   {
      // Do something with this.Engine
   }
}
```

[目录](./../Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[StyletIoC-Keys](./StyletIoC-Keys.md)