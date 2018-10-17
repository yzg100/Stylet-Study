The ViewManager is one of Stylet's core components, and is responsible for taking a ViewModel, and locating the correct View for it. It then instantiates that View, and binds it to the ViewModel.

This article will teach you how this process works, and how to modify it.

---
><font color="#63aebb" face="微软雅黑">ViewManager 是 Stylet 的核心组件之一，负责获取 ViewModel，并为其定位正确的 View。然后实例化 View，并绑定 ViewModel。

>本文将教你如何处理此过程以及如何对其进行修改。</font>

Locating a View - 定位 View
---------------

The default view-location strategy is pretty simple: replace the string "ViewModel" with the string "View" in the type and namespace of the ViewModel, then search all configured assemblies for a type with that name.

That's it.

'Configured Assemblies' means all assemblies in the ViewAssemblies property of the ViewManager. If you want to search more assemblies, add them to this list (see further down for an example).

This process is controlled by ViewManager.LocateViewForModel, so if you want to modify it, go ahead and subclass `ViewManager` - see the section at the bottom of this article.

---
><font color="#63aebb" face="微软雅黑">默认的视图定位策略非常简单:在视图模型的类型和名称空间中，用字符串 “View” 替换字符串 “ViewModel” ，然后搜索所有配置好的程序集，寻找具有该名称的类型。

>“Configured Assemblies” 表示 ViewManager 的 ViewAssemblies 属性中的所有程序集。如果要搜索更多程序集，请将它们添加到此列表中（参见下面的示例）。

>这个过程是由 ViewManager.LocateViewForModel 控制的，所以如果你想修改它，请继承 `ViewManager` - 请参阅本文底部。</font>

Creating a View - 创建 View
---------------

Once the view type has been identified, it's instantiated by calling a factory method supplied to the ViewManager. The bootstrapper sets this up to call into your IoC container by default. `InitializeComponent` is then called, which means you can delete your codebehind if you want.

This is handled by `ViewManager.CreateViewForModel`, if you fancy overriding it.

---
><font color="#63aebb" face="微软雅黑">一旦识别出视图类型，就可以通过调用提供给 ViewManager 的工厂方法来实例化它。默认情况下，bootstrapper 将其设置为调用 IoC 容器。然后调用 `InitializeComponent`，这意味着你可以删除你的后台代码。

>如果你想要重写它 `ViewManager.CreateViewForModel` 由此处理。</font>

Binding the View to its ViewModel - 将 ViewModel 绑定到 View
---------------------------------

Binding a ViewModel and its corresponding View is pretty straightforward:
 - The View's DataContext is set to the ViewModel.
 - The View's ActionTarget is set to the ViewModel (see [Actions](./Actions.md)).
 - If the ViewModel implements `IViewAware`, its `AttachView` method is called, and the View passed (see [Screens and Conductors - 屏幕和指挥](./Screens-and-Conductors.md)).

---
><font color="#63aebb" face="微软雅黑">绑定 ViewModel 及其对应的 View 非常简单：
>- View 的 DataContext 属性设置为 ViewModel。
>- View 的 ActionTarget 被设置为 ViewModel (参见[Actions](./Actions.md))。
>- 如果 ViewModel 实现了 `IViewAware`，则调用它的 `AttachView` 方法，并传递视图(参见[Screens and Conductors - 屏幕和指挥](./Screens-and-Conductors.md))。</font>

Dealing with `View.Model` Changes - 处理 `View.Model` 变更
---------------------------------

The ViewManager is also responsible for reacting to changes of a `ContentControl`'s `View.Model` attached property - see  [ViewModel First - 视图模型优先](./ViewModel-First.md).

It will grab the new value of the attached property (which should have been bound to a ViewModel instance), and use `CreateViewForModel` and `BindViewToModel` to create a new configured View instance, before setting it as the `ContentControl`'s content.

---
><font color="#63aebb" face="微软雅黑">ViewManager 还负责对 `ContentControl` 的 `View.Model` 附加属性的更改做出反应 - 请参阅 [ViewModel First - 视图模型优先](./ViewModel-First.md)。

>它将获取附加属性的新值(绑定到 ViewModel 实例)，并使用 `CreateViewForModel` 和 `BindViewToModel` 创建一个新配置的视图实例，设置为 `ContentControl` 的内容。</font>

Configuring the ViewManager - 配置 ViewManager
---------------------------

The ViewManager is configurable. To configure it, grab an instance of it in your `Configure` method:

---
><font color="#63aebb" face="微软雅黑">ViewManager 是可配置的，可以在 `Configure` 方法中获取它的实例进行配置：</font>

```csharp
protected override void Configure()
{
    var viewManager = this.Container.Get<ViewManager>();
    // ...
}
```

Once you've got it, there are several things you can configure on it:

 - `ViewSuffix`: the suffix of your View class names. Defaults to "View".
 - `ViewModelSuffix`: the suffix of your ViewModel class names. Defaults to "ViewModel".
 - `ViewAssemblies`: which assemblies to scan for views. Defaults to the assembly containing your bootstrapper.
 - `NamespaceTransformations`: this is a dictionary of `from -> to` substitutions, and allows your Views and ViewModels to live in different namespaces. If your Views live in "Foo.Frontend.Views" and your ViewModels live in "Foo.FrontendLogic.ViewModels", you could add an entry to this dictionary with the key "Foo.Frontend" and the value "Foo.FrontendLogic". Note that you have to include all root namespaces: you couldn't have used a transformation from "Frontend" to "FrontendLogic" here.

---
><font color="#63aebb" face="微软雅黑">一旦你得到它，你可以配置它们：
>- `ViewSuffix`: View 类名称后缀。默认为 "View"。
>- `ViewModelSuffix`: ViewModel 类名称后缀。默认为 "ViewModel"。
>- `ViewAssemblies`: 要扫描视图的程序集。默认为包含 bootstrapper 的程序集。
>- `NamespaceTransformations`: 这是 `from -> to` 替换字典，允许你的 Views 和 ViewModel 存在于不同的命名空间中。如果你的视图在 “Foo.Frontend.Views” 而 ViewModel 在 “Foo.FrontendLogic.ViewModels” 中，则可以使用键 “Foo.Frontend” 和值 “Foo.FrontendLogic” 向该字典添加一项。请注意，你必须包含所有根命名空间：你不能使用从 “Frontend” 到 “FrontendLogic” 的转换。</font>


Creating your own ViewManager - 创建自己的 ViewManager
-----------------------------

When Stylet needs the ViewManager, it retrieves an instance of `IViewManager` from the IoC container. This means that you can create an implementation of IViewManager you want and register it with the IoC container, and Stylet will pick it up and use it.

A very simplistic example:

---
><font color="#63aebb" face="微软雅黑">当 Stylet 需要 ViewManager 时，它会从 IoC 容器中检索 `IViewManager` 实例。这意味着你可以创建所需的 `IViewManager` 实现并将其注册到 IoC 容器中，Stylet 将获取并使用它。

>例子：</font>

```csharp
// CustomViewManager.cs

public class CustomViewManager : ViewManager
{
   public CustomViewManager(ViewManagerConfig config)
      : base(config)
   { }

   public override UIElement CreateAndSetupViewForModel(object model)
   {
      // This dummy application only has one view, and it's this one :)
      // 这个虚拟应用程序只有一个视图，就是这个视图 :) 
      return new SingletonView();
   }
}

// Bootstrapper.cs

public class Bootstrapper : Bootstrapper<MyRootViewModel>
{
   protected override void ConfigureIoC(IStyletIoCBuilder builder)
   {
      builder.Bind<IViewManager>().To<CustomViewManager>();
   }
}
```

And that's it!

A more complete example can be found in the [`Stylet.Samples.OverridingViewManager` sample](https://github.com/canton7/Stylet/tree/master/Samples/Stylet.Samples.OverridingViewManager). 

---
><font color="#63aebb" face="微软雅黑">就是这样！

>可以在 [`Stylet.Samples.OverridingViewManager` sample](https://github.com/canton7/Stylet/tree/master/Samples/Stylet.Samples.OverridingViewManager) 示例中找到更完整的示例。</font>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Listening to INotifyPropertyChanged - 监听 INotifyPropertyChanged](./Listening-to-INotifyPropertyChanged.md)