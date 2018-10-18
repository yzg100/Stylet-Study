Caliburn.Micro came with a static service locator called `IoC`. This let you access the IoC container from anywhere in your code, like this:

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 带有一个静态服务定位 IoC。这使你可以从代码中的任何位置访问IoC容器，如下：</font>

```csharp
var vm = IoC.Get<MyDialogViewModel>();
this.windowManager.ShowDialog(vm);
```

Stylet doesn't include an equivalent, and with good reason: I don't want to encourage people to write such horrible code. The Service Locator pattern is frequently termed an anti-pattern. Every class now has a dependency on `IoC` (instead of the *actual* classes it depends on), and you can't tell just by looking at the class's constructor what its dependencies are: you have to scour through the code for calls to `IoC.Get`. 

`IoC` was also used internally within Caliburn.Micro to get around some poor design choices. These have been re-architected in Stylet, and so `IoC` is no longer required internally.

If you really *really* need `IoC` back (and it's a deal-breaker), then you can write your own very easily. First create this static `IoC` class:

---
><font color="#63aebb" face="微软雅黑">Stylet 不包含类似的代码，而且有充分的理由:我不想鼓励人们编写如此糟糕的代码。服务定位器模式经常被称为反模式。现在每个类都依赖于 `IoC` (而不是它所依赖的*实际*类)，你不能仅仅通过查看类的构造函数就知道它的依赖关系是什么:你必须遍历代码来调用 `IoC.Get`。

>`IoC` 在 Caliburn.Micro 内部还使用了一些糟糕的设计。这些已在 Stylet 中已重新设计，因此内部不再需要 `IoC`。

>如果你真的真的需要 `IoC`，那么你可以自己编写。首先创建这个静态 `IoC` 类：</font>

```csharp
public static class IoC
{
    public static Func<Type, string, object> GetInstance = (service, key) => { throw new InvalidOperationException("IoC is not initialized"); };

    public static Func<Type, IEnumerable<object>> GetAllInstances = service => { throw new InvalidOperationException("IoC is not initialized"); };

    public static Action<object> BuildUp = instance => { throw new InvalidOperationException("IoC is not initialized"); };

    public static T Get<T>(string key = null)
    {
        return (T)GetInstance(typeof(T), key);
    }

    public static IEnumerable<T> GetAll<T>()
    {
        return GetAllInstances(typeof(T)).Cast<T>();
    }
}
```

Then wire it into your bootstrapper like this:

---
><font color="#63aebb" face="微软雅黑">然后放到你的 bootstrapper 中，如下:</font>

```csharp
protected override void Configure()
{
   IoC.GetInstance = this.Container.Get;
   IoC.GetAllInstances = this.Container.GetAll;
   IoC.BuildUp = this.Container.BuildUp;
}
```

[目录](./Index.md)&nbsp;&nbsp;