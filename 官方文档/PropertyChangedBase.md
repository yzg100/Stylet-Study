PropertyChangedBase is base class for types implementing INotifyPropertyChanged, and provides methods for raising PropertyChanged notifications.

---
><font color="#63aebb" face="微软雅黑">PropertyChangedBase 是实现 INotifyPropertyChanged 类型的基类，并提供了引发 PropertyChanged 通知的方法。</font>

Raising Notifications - 引发通知
---------------------

There are a number of ways to raise PropertyChanged notifications, depending on what exactly you want to do.

The most common case is having a property raise a notification each time it's assigned to. PropertyChangedBase provides a nice utility method to help: SetAndNotify. It takes, by reference, a field, and a value to assign to the field. If the field's value != the value, the assignment happens, and a PropertyChanged notification is raised. For example:

---
><font color="#63aebb" face="微软雅黑">有很多方法可以引发 PropertyChanged 通知，具体取决于你想要做什么。

>最常见的情况是每次给属性赋值时都引发通知。PropertyChangedBase 提供了一个很好的实用方法：SetAndNotify。它通过检查获取字段和分配给字段的值。如果字段的值与新值不相等，则赋值，并引发 PropertyChanged 通知。例如：</font>

```csharp
class MyClass : PropertyChangedBase
{
   private string _name;
   public string Name
   {
      get { return this._name; }
      set { SetAndNotify(ref this._name, value); }
   }
```

If you want to raise a PropertyChanged notification for a property other than the current one, there are a few ways of achieving that, too (depending on whether you're using C#6, or below).
The preferred C# 6 way is to use `nameof()`, as that is very cheap and provides compile-time safety.
If you're using C# 5 or below, you can use an expression: this is slower, but also gives compile-time safety.
If you *really* want, you can use a raw string as well.
See below:

---
><font color="#63aebb" face="微软雅黑">如果你想为别的属性引发 PropertyChanged 通知，也可以通过几种方法实现（取决于你使用的是 C＃6还是以下版本）。首选的 C＃6 方式使用 `nameof()`，因为它非常方便并且提供编译时的安全性。如果你使用 C＃5 或更低版本，你可以使用一个表达式：这个速度较慢，但​​也提供了编译时的安全性。你也可以使用原始字符串。如下：</font>

```csharp
class MyClass : PropertyChangedBase
{
   public string Name { get; private set; }

   public void RaiseNameChangedExpression()
   {
      // Preferred if you're using C# 6, as it provides compile-time safety
      this.NotifyOfPropertyChange(nameof(this.Name));

      // Preferred for C# 5 and below, as it provides compile-time safety
      this.NotifyOfPropertyChange(() => this.Name);

      // Not preferred, but is very occasionally necessary
      this.NotifyOfPropertyChange("Name");
   }
}
```

Dispatching Events - 事件调度
------------------

By default, PropertyChanged events are raised on the current thread (and WPF takes care of dispatching them to the UI thread). If you do want to change this, however, you can! PropertyChangedBase has a property called PropertyChangedDispatcher, which has an `Action<Action>`, and defaults to Execute.DefaultPropertyChangedDispatcher (which you can assign) (which has the value `a => a()`).

If you want to change this to execute on the UI thread, you can do the following.

To change for all instances of PropertyChangedBase:

---
><font color="#63aebb" face="微软雅黑">默认情况下，在当前线程上引发 PropertyChanged 事件（WPF 负责将它们分派到 UI 线程）。PropertyChangedBase 有一个名为 PropertyChangedDispatcher 的属性，它有一个 `Action<Action>` 参数，默认为 Execute.DefaultPropertyChangedDispatcher 可以赋值 `a => a()`。

>如果要将其更改为在UI线程上执行，则可以执行以下操作。

>要更改 PropertyChangedBase 的所有实例：</font>

```csharp
class Bootstrapper : Bootstrapper<MyRootViewModel>
{
   public override void Configure()
   {
      base.Configure();
      Execute.DefaultPropertyChangedDispatcher = Execute.OnUIThread;
   }
}
```

To change for just once instance of PropertyChangedBase:

---
><font color="#63aebb" face="微软雅黑">仅更改实例 PropertyChangedBase 一次:</font>

```csharp
class MyClass : PropertyChangedBase
{
   public MyClass()
   {
      this.PropertyChangedDispatcher = Execute.OnUIThread;
   }
}
```

Use with PropertyChanged.Fody - 与 PropertyChanged.Fody 一起使用
-----------------------------
[PropertyChanged.Fody](https://github.com/Fody/PropertyChanged) is a fantastic package, which injects code at compile-time to automatically raise PropertyChanged notifications for your properties, allowing you to write very concise code. It will also figure out dependencies between your properties and raise notifications appropriately for example:

---
><font color="#63aebb" face="微软雅黑">[PropertyChanged.Fody](https://github.com/Fody/PropertyChanged) 是一个很棒的包，它在编译时注入代码以自动为你的属性引发 PropertyChanged 通知，允许你编写非常简洁的代码。它还将找出属性之间的依赖关系并适当地引发通知，例如：</font>

```csharp
class MyClass : PropertyChangedBase
{
   public string FirstName { get; private set; }
   public string LastName { get; private set; }
   public string FullName { get { return String.Format("{0} {1}", this.FirstName, this.LastName); } }

   public void SomeMethod()
   {
      // PropertyChanged notifications are automatically raised for both FirstName and FullName

      //自动引发 FirstName 和 FullName 的 PropertyChanged 通知
      this.FirstName = "Fred";
   }
```

PropertyChangedBase also takes care to integrate with `Fody.PropertyChanged`. Notifications raised by `Fody.PropertyChanged` are raised using the `PropertyChangedDispatcher`.

Therefore you do not need to do anything special in order to use `Fody.PropertyChanged` with any subclass of `Screen`, `ValidatingModelBase`, or `PropertyChangedBase`.

---
><font color="#63aebb" face="微软雅黑">PropertyChangedBase 还与 `Fody.PropertyChanged` 进行了集成。 `Fody.PropertyChanged` 使用 `PropertyChangedDispatcher` 引发的通知。

>对于 `Screen`、`ValidatingModelBase` 或 `PropertyChangedBase` 的任何子类，你不需要做任何特殊的事就可以使用 `Fody.PropertyChanged`。</font>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Execute: Dispatching to the UI Thread - 执行:UI线程调度](./Execute.md)