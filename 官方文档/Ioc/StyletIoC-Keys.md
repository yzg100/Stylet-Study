So far I've been saying that StyletIoC holds a mapping of `service -> implementation type(s)`.

That's actually a little lie. In fact, the mapping is `[service, key] -> implementation type(s)`, where `key` is an arbitrary string which you supply.

This concept of keys allows you to have, say, two implementations of `IVehicle`, but things requiring an `IVehicle` can specify, indirectly, which implementation they're after.

There are many ways to specify this key, both when you create the registration, and when you request an instance of a service from StyletIoC.

---
><font color="#63aebb" face="微软雅黑">到目前为止，我一直在说 StyletIoC 拥有映射 `service -> implementation type(s)`。

>这实际上是一个小谎言。实际上，映射是 `[service, key] -> implementation type(s)`，`key` 是你提供的任意字符串。

>Key 的概念允许你有两个 IVehicle 的实现，但需要 IVehicle 可以间接的指定是那个实现。

>在创建注册和从 StyletIoC 请求服务实例时，有很多方法可以指定 Key。</font>

Specifying The Key Associated With A Type - 指定与类型关联的 Key
-----------------------------------------

Keys can be specified in a few ways.

---
><font color="#63aebb" face="微软雅黑">可以用几种方式指定 Key。

>你可以使用 `[Inject(Key = "mykey")] 属性修饰指定要与类型关联的 Key，例如：</font>

You can specify the key to be associated with a type by using the `[Inject(Key = "mykey")]` attribute on the type itself, for example:

```csharp
[Inject(Key = "old")]
class OldBanger : IVehicle { ... }
```

You can also specify it when binding the type, using the `WithKey` method. This overrides the key from the attribute, if any:

```csharp
builder.Bind<IVehicle>().To<HotHatchback>().WithKey("new");
builder.Bind<IVehicle>().To(container => new OldBanger()).WithKey("old");
```

Retrieving A Type Using Its Key - 使用 Key 检索类型
-------------------------------

All of the different injection methods methods support specifying the key to use when deciding which type to inject.

When fetching instances directory using `IContainer.Get`, you can pass the key to use:

---
><font color="#63aebb" face="微软雅黑">所有不同的注入方法都支持在决定注入哪种类型时指定要使用的 Key。

>使用获取实例目录时 `IContainer.Get`，可以传递 Key 使用：</font>

```csharp
ioc.Get<IVehicle>("old");
```

When using constructor injection, you can specify the key use using the `[Inject(Key = "key")` attribute:

---
><font color="#63aebb" face="微软雅黑">在使用构造函数注入时，可以使用 `[Inject(Key = "key")` 修饰指定 Key 的使用：</font>

```csharp
class CarOwner
{
   // IVehicle will be a HotHatchback, as that was registered with the key "new"
   // IVehicle 将是 HotHatchback，因为它使用 Key "new" 注册 
   public CarOwner([Inject(Key = "new") IVehicle vehicle) { ... }
}
```

Similarly, you can use the `[Inject(Key = "key")]` syntax when using property injection:

---
><font color="#63aebb" face="微软雅黑">同样，在使用属性注入时，可以使用 `[Inject(Key = "key")]` 语法：</font>

```csharp
class CarOwner
{
   [Inject(Key = "old")]
   public IVehicle Vehicle { get; set; }
}
```

Keys and Collections - Key 与 集合
--------------------

You can also use keys with services that have more than one implementation, for example:

---
><font color="#63aebb" face="微软雅黑">你还可以将 Key 与具有多个实现的服务一起使用，例如：</font>

```csharp
builder.Bind<IVehicle>().To<HotHatchback>().WithKey("new");
builder.Bind<IVehicle>().To<Roadster>().WithKey("new");
builder.Bind<IVehicle>().To<OldBanger>().WithKey("old");

// 然后...

class NewCarGarage
{
   public NewCarGarage([Inject("new")] IEnumerable<IVehicle> vehicles) { ... }
}

class OldCarGarage
{
   public OldCarGarage([Inject("old")] IEnumerable<IVehicle> vehicles { ... }
}

// 或者...

var newVehicles = ioc.GetAll<IVehicle>("new");
```

[目录](./../Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[StyletIoC-Factories - StyletIoC 工厂](./StyletIoC-Factories.md)