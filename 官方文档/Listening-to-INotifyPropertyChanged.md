It may be easy to implement `INotifyPropertyChanged`, but it's often a bit of a pain to watch such an object for `PropertyChanged` notifications - you need to register an event handler, have it check the property name to see whether it's the one you expect, and unregister the event handler when you're done.

This is such a common problem that Stylet provides a couple of methods to make life easier.

---
><font color="#63aebb" face="微软雅黑">实现 `INotifyPropertyChanged` 很容易，但要监视对象获得 `PropertyChanged` 通知通常有点痛苦  -你需要注册一个事件处理程序，让它检查属性名称以确定它是否是你期望的名称，并在完成之后将事件处理程序取消注册。 

>这是一个非常常见的问题，因此 Stylet 提供了一些方法来简化工作。</font>

INotifyPropertyChanged.Bind
---------------------------

This is the simplest way of subscribing to PropertyChanged events, and it does so using a strong reference to the subscriber (as normal events do). This means that if you plan on being released while the thing you're subscribing to is still alive, you'll have to remember to unsubscribe.

Usage is pretty simple. Assuming there's an object of the form:

---
><font color="#63aebb" face="微软雅黑">这是订阅PropertyChanged事件的最简单方法，它使用对订阅者的强引用（如普通事件那样）。这意味着如果你计划在你订阅的内容仍然存在时被释放，你将不得不记得取消订阅。

>用法很简单。假设有一个对象：</font>

```csharp
// Can be any implementation of INotifyPropertyChanged - I'm using PropertyChangedBase as it makes the example shorter
// 可以是 INotifyPropertyChanged 的任何实现 - 我正在使用 PropertyChangedBase，因为它使示例更短的
class Model : PropertyChangedBase
{
   private string _stringProperty;
   public string StringProperty
   {
      get { return this._stringProperty; }
      set { SetAndNotify(ref this._stringProperty, value); }
   }
}
```

and you want to be notified every time the `StringProperty` property changes. You do so like this:

---
><font color="#63aebb" face="微软雅黑">并且你希望每次 `StringProperty` 属性更改时收到通知。这样做：</font>

```csharp
var model = new Model();
// ... 
model.Bind(x => x.StringProperty, (sender, eventArgs) => Debug.WriteLine(String.Format("New value for property {0} on {1} is {2}", eventArgs.PropertyName, sender, eventArgs.NewValue)));
```

The `x => x.StringProperty` bit is a way of specifying which property you want to observe in a type-safe way. The `x` can be named anything you like, and intellisense will prompt you with a list of properties when you get as far as `x => x.`.

The `(propertyName, newValue) => Debug.WriteLine(String.Format("New value is {0}", newValue))` bit is called every time that property changes.

The `Bind` method actually returns an implementation of `IEventBinding`, which has a single `Unbind` method. To remove a binding, call that method. For example:

---
><font color="#63aebb" face="微软雅黑">`x => x.StringProperty` 是一种以类型安全的方式指定要观察的属性的方法。`x` 可以命名为任何你喜欢的，它会以智能感方式提示你属性列表 `x => x.`。

>每当属性改变时都会调用 `(propertyName, newValue) => Debug.WriteLine(String.Format("New value is {0}", newValue))`。

>`Bind` 方法返回 `IEventBinding` 的实现，它有一个 `Unbind` 方法。要解除绑定，请调用该方法。例如：</font>

```csharp
var model = new Model();
// ... 
var binding = model.Bind(x => x.StringProperty, (sender, eventArgs) => Debug.WriteLine(String.Format("New value for property {0} on {1} is {2}", eventArgs.PropertyName, sender, eventArgs.NewValue)));
// ...
binding.Unbind();
```

INotifyPropertyChanged.BindWeak
-------------------------------

Normally, when you subscribe to an event, the thing receiving event notifications will live at least as long as the thing publishing the events, since the thing publishing events ends up with a reference to the thing receiving the event notifications.

This is often undesirable. For example, if you've got a ViewModel that wants to watch for PropertyChanged events on some service that it depends on.

Stylet provides an extension method on INotifyPropertyChanged called `BindWeak`, which is very similar to `Bind`, except that it creates a weak binding. The syntax is the same as `Bind`, so I won't repeat it here.

Note that it's not possible bind every delegate in a weak manner. Delegates which capture local variables will often fail. This is discussed in more detail below.

---
><font color="#63aebb" face="微软雅黑">通常，当你订阅事件时，接收事件通知的对象的生命周期至少与发布事件的对象一样长，因为发布事件的对象最终会引用接收事件通知的对象。

>这通常是不合需要的。例如，如果你有一个想要在其依赖的某些服务上监视 PropertyChanged 事件的 ViewModel。

>Stylet 在 INotifyPropertyChanged 上提供了一个名为 `BindWeak` 的扩展方法，它与 `Bind` 非常相似，只是它创建了一个弱绑定。语法与 `Bind` 相同，在此不再赘述。

>注意，不能以弱方式绑定每个委托。捕获局部变量的委托通常会失败。下面将对此进行更详细的讨论。</font>

Technical: Weak Event Subscriptions - 弱事件订阅
-----------------------------------

I'm going to gloss over some of the finer points of delegates, but in basic terms, when you subscribe to an event, you create a new delegate instance, and pass it to the thing owning the event. A delegate consists of (basically) two things: A method to call (the `Method`) property, and an instance to call that method of (the `Target` property).

If you create a delegate which points to an instance method on your class, everything's nice and simple:

---
><font color="#63aebb" face="微软雅黑">我将略去委托的一些细节，但从基本的角度来说，当你订阅一个事件时，你将创建一个新的委托实例，并将其传递给拥有该事件的对象。委托由两部分组成:要调用的方法( `Method` )属性和要调用的方法( `Target `)属性。

>如果创建一个指向类的实例方法的委托，一切都很简单：</font>

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      model.PropertyChanged += new PropertyChangedEventHandler(this.PropertyChangedHandler);
      // or, more concisely
      //简洁的
      model.PropertyChanged += this.PropertyChangedHandler;
   {

   public void PropertyChangedHandler(object sender, PropertyChangedEventArgs e)
   {
      // ...
   }
}
```

In that case, a new delegate is creates with its `Target` set to the `MyClass` instance, and its `Method` set to a `MethodInfo` representing your `PropertyChangedHandler` method.

The `model` instance then becomes the owner of that delegate. This means that the `model` instance owns a delegate which owns the `MyClass` instance, meaning that the `MyClass` instance cannot by released until the `model` instance is.

Things start getting a bit more complex when anonymous delegates / lambdas come into play, for example:

---
><font color="#63aebb" face="微软雅黑">在这种情况下，创建一个新的委托，`Target` 设置为 `MyClass` 实例，`Method` 设置为 `MethodInfo` 表示 `PropertyChangedHandler` 方法。

>`model` 实例成为该委托的所有者。意味着 `model` 实例拥有一个拥有 `MyClass` 实例的委托，意味着 `MyClass` 实例在 `model` 实例被释放之前不会被释放。

>当匿名委托 / lambdas 发挥作用时，事情开始变得更复杂，例如：</font>

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      model.PropertyChanged += delegate { Debug.WriteLine("Hi"); };
      // Or, using lambas (preferred)
      model.PropertyChanged += (o, e) => Debug.WriteLine("Hi");
   }
}
```

Here, the C# compiler has to create a new, special method on your class, which looks something like this:

---
><font color="#63aebb" face="微软雅黑">在这里，C＃ 编译器必须在你的类上创建一个新的特殊方法，像这样：</font>

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      model.PropertyChanged += new PropertyChangedEventHandler(this.<.ctor>b__0);
   }

   [CompilerGenerated]
   private void <.ctor>b__0(object sender, PropertyChangedEventArgs e)
   {
      Debug.WriteLine("Hi");
   }
}
```

(Note the use of an "unspeakable" method name - one containing characters (`<` and `>`) which aren't valid in C#, but are in the CLR).

This starts getting even more complex if we have an anonymous delegate/lambda which captures a local variable. Here, the C# compiler needs to generate a whole new embedded class, which keeps a reference to that variable. For example:

---
><font color="#63aebb" face="微软雅黑">(注意使用 “unspeakable” 的方法名 - 包含字符( `<` 和 `>` )的方法名，这些字符在 C# 中无效，但在 CLR 中有效)。

>如果我们有一个匿名委托 / lambda(它捕获一个局部变量)，这将变得更加复杂。C# 编译器需要生成一个全新的内嵌类，它保留对该变量的引用。例如:</font>

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      string test = "test";
      model.PropertyChanged += (o, e) => Debug.WriteLine(test);
   }
}
```

Is compiled into something which looks a bit like:

---
><font color="#63aebb" face="微软雅黑">被编译成这样：</font>

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      MyClass.<>c__DisplayClass1 c__DisplayClass1 = new MyClass.<>c__DisplayClass1();
      c__DisplayClass1.test = "test";
      model.PropertyChanged += new PropertyChangedEventHandler(c__DisplayClass1.<.ctor>b__0);
   }

   [CompilerGenerated]
   private sealed class <>c__DisplayClass1
   {
      public string test;
      public void <.ctor>b__0(object sender, PropertyChangedEventArgs e)
      {
         Debug.WriteLine(this.test);
      }
   }
```

Here, the PropertyChangedEventHandler delegate that's created will have the instance of `<>c__DisplayClass` as the value of its `Target` property.

This means, that when `MyClass`'s constructor returns, the *only* thing with a reference to that `<>c__DisplayClass1` instance is the delegate. The lifecycle of the `<>c__DisplayClass1` instance is now entirely independent of the `MyClass` instance.

The way of implementing weak events is to make the delegate's `Target` property a `WeakReference` in some way - usually by having it point to an intermediate class, which in turn has a `WeakReference` to the "real" Target. This means that the Target isn't retained by the delegate.

If this Target is a compiler-generated inner class, then nothing else will hold a reference to it, other than the `WeakReference` we created. This means that this inner class is collected straight away, and so the delegate will never be called.

Because of this, `BindWeak` will throw an exception if the delegate given to it has a `Target` which has the `CompilerGenerated` attribute.

---
><font color="#63aebb" face="微软雅黑">这里创建的 PropertyChangedEventHandler 委托将具有 `<>c__DisplayClass` 作为 `Target` 属性的值。

>当 `MyClass` 构造函数返回时，唯一引用 `<>c__DisplayClass1` 实例的是委托。`<>c__DisplayClass1` 实例的生命周期现在完全独立于 `MyClass` 实例。

>实现弱事件的方法是让委托的 `Target` 属性成为 `WeakReference` - 通过让它指向一个中间类，而中间类又有一个 `WeakReference` 指向 “真正的” 目标。这意味着目标不被委托保留。

>如果这个目标是编译器生成的内部类，那么除了我们创建的 `WeakReference` 之外，就没有其他引用了。这意味着直接收集此内部类，因此永远不会调用委托。

>因此，`BindWeak` 如果赋予它的委托 `Target` 具有 `CompilerGenerated` 特性，则抛出异常。</font>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Design Mode Support - 设计模式支持](./Design-Mode-Support.md)