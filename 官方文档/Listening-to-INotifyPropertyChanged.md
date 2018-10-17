It may be easy to implement `INotifyPropertyChanged`, but it's often a bit of a pain to watch such an object for `PropertyChanged` notifications - you need to register an event handler, have it check the property name to see whether it's the one you expect, and unregister the event handler when you're done.

This is such a common problem that Stylet provides a couple of methods to make life easier.

INotifyPropertyChanged.Bind
---------------------------

This is the simplest way of subscribing to PropertyChanged events, and it does so using a strong reference to the subscriber (as normal events do). This means that if you plan on being released while the thing you're subscribing to is still alive, you'll have to remember to unsubscribe.

Usage is pretty simple. Assuming there's an object of the form:

```csharp
// Can be any implementation of INotifyPropertyChanged - I'm using PropertyChangedBase as it makes the example shorter
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

```csharp
var model = new Model();
// ... 
model.Bind(x => x.StringProperty, (sender, eventArgs) => Debug.WriteLine(String.Format("New value for property {0} on {1} is {2}", eventArgs.PropertyName, sender, eventArgs.NewValue)));
```

The `x => x.StringProperty` bit is a way of specifying which property you want to observe in a type-safe way. The `x` can be named anything you like, and intellisense will prompt you with a list of properties when you get as far as `x => x.`.

The `(propertyName, newValue) => Debug.WriteLine(String.Format("New value is {0}", newValue))` bit is called every time that property changes.

The `Bind` method actually returns an implementation of `IEventBinding`, which has a single `Unbind` method. To remove a binding, call that method. For example:

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

Technical: Weak Event Subscriptions
-----------------------------------

I'm going to gloss over some of the finer points of delegates, but in basic terms, when you subscribe to an event, you create a new delegate instance, and pass it to the thing owning the event. A delegate consists of (basically) two things: A method to call (the `Method`) property, and an instance to call that method of (the `Target` property).

If you create a delegate which points to an instance method on your class, everything's nice and simple:

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      model.PropertyChanged += new PropertyChangedEventHandler(this.PropertyChangedHandler);
      // or, more concisely
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