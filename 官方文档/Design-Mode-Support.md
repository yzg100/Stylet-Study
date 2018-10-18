Introduction - 介绍
------------

"Design Mode" or "Design-time" refers to when you project is loaded into the Visual Studio XAML Designer, or Expression Blend, and you're being shown a rendered version of your XAML. Most of the time, the designer won't try and evaluate any of your bindings or provide any IntelliSense for them. However, with a bit of configuration, you can get lovely IntelliSense, and display some dummy values from your ViewModel in your View.

Stylet has some basic support for Design Mode. This article documents it, and provides instruction on how to use it and make use of existing XAML features to enhance your design-time experience.

All of the examples shown here are available "ready to run" in the [DesignMode sample project](https://github.com/canton7/Stylet/tree/master/Samples/Stylet.Samples.DesignMode).

---
><font color="#63aebb" face="微软雅黑">“设计模式” 或 “设计时” 是指将项目加载到 Visual Studio XAML 设计器或 Expression Blend 时，并且你正在显示 XAML 的渲染版本。大多数情况下，设计人员不会尝试评估任何绑定或为其提供任何智能感知。但是，通过一些配置，你可以获得可爱的智能感知，并在 View 中显示 ViewModel 中的一些虚拟值。

>Stylet 对设计模式有一些基本的支持。本文对其进行了记录，并提供了有关如何使用它的说明，并利用现有的XAML功能来增强你的设计时体验。

>此处显示的所有示例都可在 [DesignMode sample project](https://github.com/canton7/Stylet/tree/master/Samples/Stylet.Samples.DesignMode) 中运行。</font>


IntelliSense only, no bindings - 仅智能感知，无绑定
------------------------------

This is the most basic technique, and requires very little extra work on your part. You'll get IntelliSense for your bindings (at least in Visual Studio 2013 and above), but you won't see any dummy data from your ViewModel shown in your View.

First, you'll need the following declarations in the root of your View. If you've created a UserControl in Visual Studio 2013, these will have been added by default.

---
><font color="#63aebb" face="微软雅黑">这是最基本的技巧，你几乎不需要额外的工作。你将获得绑定的智能感知(至少在 Visual Studio 2013 及以上版本中)，但是你不会看到视图模型中显示的任何虚拟数据。

>首先，你需要在 View 的根目录中进行以下声明。如果你在 Visual Studio 2013 中创建了 UserControl，则默认情况下会添加这些 UserControl。</font>

```
xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
mc:Ignorable="d" 
```

You'll also need to add a namespace for your ViewModels:

---
><font color="#63aebb" face="微软雅黑">你还需要为 ViewModel 添加命名空间：</font>

```
xmlns:vms="clr-namespace:DesignMode.ViewModels"
```

Once you've got this, you need one extra line of magic. This tells the XAML designer that the `DataContext` for this View is your `SampleViewModel`, and that binding IntelliSense should use the properties on this:

---
><font color="#63aebb" face="微软雅黑">一旦你得到它，你需要一行额外的魔术。告诉 XAML 设计器 View 的 `DataContext` 是 `SampleViewModel`，智能感知应该使用这个属性绑定:</font>

```
d:DataContext="{d:DesignInstance vms:SampleViewModel}"
```

Putting it all together, you'll end up with something like this:

---
><font color="#63aebb" face="微软雅黑">把它们放在一起，你会得到这样的内容:</font>

```xml
<UserControl x:Class="DesignMode.Views.SampleView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             mc:Ignorable="d"
             xmlns:vms="clr-namespace:DesignMode.ViewModels"
             d:DataContext="{d:DesignInstance vms:SampleViewModel}">
   ...
</UserControl>
```

Intellisense and Dummy Data, letting the Designer instantiate the ViewModel - 智能感知和虚拟数据，让设计器实例化 ViewModel
---------------------------------------------------------------------------

This technique is very similar to the previous one, except we let the XAML designer instantiate the ViewModel for us. The designer will use this ViewModel instance to grab dummy data for your bindings from.

In order for the designer to be able to do this, the ViewModel must have a parameterless constructor. This is both a blessing and a curse. The good side is that it gives you a nice place to inject some dummy data into the ViewModel's properties for the designer to use. The bad side is that your ViewModel now contains code which is only ever used by the designer...

**Note** that there's no access to the IoC container at design-time (and anyone requesting access will be taken around the back and... dealt with). Therefore if your ViewModel has any dependencies, they won't be available at design-time. Normally this isn't a problem: only properties are accessed (no methods are called), and so you shouldn't really be doing anything which requires access to one of your dependencies. Just bear it in mind.

A sample ViewModel written to support design mode in this way may look like this:

---
><font color="#63aebb" face="微软雅黑">这项技术与前一种技术非常相似，只是我们让 XAML 设计器为我们实例化 ViewModel。设计人员将使用此 ViewModel 实例从中获取绑定的虚拟数据。

>为了使设计人员能够执行此操作，ViewModel 必须具有无参数构造函数。这既是福也是祸，好的是它为你提供了一个位置，可以将虚拟数据注入到 ViewModel 的属性中供设计人员使用。不好的是 ViewModel 现在包含的代码只有设计人员才能使用...

>注意，在设计时无法访问 IoC 容器（请求访问的任何人都将在后面进行处理...）。因此，如果你的 ViewModel 具有任何依赖项，则它们将无法在设计时使用。通常这不是问题：只访问属性（不调用任何方法），因此你不应该做任何需要访问依赖项的事情。请记住它。

>以这种方式编写以支持设计模式的示例 ViewModel 如下：</font>

```csharp
public class SampleViewModel
{
    private readonly IUserService userService;

    public string CurrentUserName { get; private set; }

    public SampleViewModel()
    {
        this.CurrentUserName = "Dummy Username";
    }

    public SampleViewModel(IUserService userService)
    {
        this.userService = userService;
        this.CurrentUserName = this.userService.CurrentUser.UserName;
    }
}
```

Note that StyletIoC will always pick the constructor with the most parameters that it can resolve, so it will also call the overload which accepts the `IUserService`. The designer on the other hand will always call the parameterless constructor.

If your ViewModel would ordinarily only have a parameterless constructor, you can make use of [Execute.InDesignMode](./Execute.md), like this:

---
><font color="#63aebb" face="微软雅黑">注意，StyletIoC 始终选择具有可解析的最多参数的构造函数，因此它也将调用接受该函数的重载 `IUserService`。而设计器始终调用无参数构造函数。

>如果你的 ViewModel 通常只有一个无参数构造函数，你可以使用 [Execute.InDesignMode](./Execute.md)，如下：</font>

```csharp
public class SampleViewModel
{
    public string SomeText { get; set; }

    public SampleViewModel()
    {
        if (Execute.InDesignMode)
            this.SomeText = "Dummy Text";
        else
            this.SomeText = "Actual Text";
    }
}
```

Either way, once you've got a ViewModel with a parameterless constructor that the designer can use, you can tell the designer to instantiate it using:

---
><font color="#63aebb" face="微软雅黑">无论哪种方式，一旦你有一个设计者可以使用的无参数构造函数的ViewModel，你可以告诉设计者使用以下方法实例化它：</font>

```
d:DataContext="{d:DesignInstance vms:SampleViewModel, IsDesignTimeCreatable=True}"
```

Or, in full - 完整的:

```xml
<UserControl x:Class="DesignMode.Views.SampleView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             mc:Ignorable="d"
             xmlns:vms="clr-namespace:DesignMode.ViewModels"
             d:DataContext="{d:DesignInstance vms:SampleViewModel, IsDesignTimeCreatable=True}">
   ...
</UserControl>
```

Intellisense and Dummy Data, using a ViewModelLocator - 智能感知和虚拟数据，使用 ViewModelLocator
-----------------------------------------------------

The previous method has a huge drawback: it requires that your ViewModel knows about design mode, and contains code that is only ever invoked at design time. This is considered by some to be a huge code smell.

An alternative approach is to use a ViewModelLocator - a class, used only at design time, which can instantiate and configure your ViewModels. This means that any design-time-only local can go in the ViewModelLocator, and kept out of the Individual Views.

If this sounds complex, bear with me. It should all make sense in a minute.

First, let's grab ourselves a sample ViewModel:

---
><font color="#63aebb" face="微软雅黑">前一种方法有一个很大的缺点：它要求你的 ViewModel 了解设计模式，并包含仅在设计时调用的代码。

>另一种方法是使用 ViewModelLocator - 一个仅在设计时使用的类，它可以实例化和配置 ViewModel。这意味着任何仅设计时的本地元素都可以放入 ViewModelLocator，并且不受单个 View 的限制。

>如果这听起来很复杂，请耐心听我说，这一切马上会变得有意义。

>示例 ViewModel：</font>

```csharp
public class SampleViewModel
{
    private readonly IUserService userService;

    public string CurrentUserName { get;set; }

    public SampleViewModel(IUserService userService)
    {
        this.userService = userService;
        this.CurrentUserName = this.userService.CurrentUser.UserName;
    }
}
``` 

Next, we need a ViewModelLocator. This is a simple class with contains one property per ViewModel we might want to access at design time:

---
><font color="#63aebb" face="微软雅黑">接下来，我们需要一个 ViewModelLocator。这是一个简单的类，每个 ViewModel 包含一个我们可能想要在设计时访问的属性：</font>

```csharp
public class ViewModelLocator
{
    public SampleViewModel SampleViewModel
    {
        get
        {
            var vm = new SampleViewModel();
            vm.CurrentUserName = "Dummy Username";
            return vm;
        }
    }
}
```

Notice how the ViewModel instantiated only when required? This is because the ViewModelLocator itself will be instantiated at runtime, but its properties will only be accessed at design time.

Next, let's add this to our application's resources, so it's available to our views:

---
><font color="#63aebb" face="微软雅黑">请注意 ViewModel 在需要时如何实例化？这是因为 ViewModelLocator 本身将在运行时实例化，但其属性只能在设计时访问。

>接下来，让我们将它添加到应用程序的资源中，让 View 可以使用它:</font>

```xml
<Application x:Class="DesignMode.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet"
             xmlns:local="clr-namespace:DesignMode">
    <Application.Resources>
        <s:ApplicationLoader>
            <s:ApplicationLoader.Bootstrapper>
                <local:Bootstrapper/>
            </s:ApplicationLoader.Bootstrapper>
            
            <local:ViewModelLocator x:Key="ViewModelLocator"/>
        </s:ApplicationLoader>
    </Application.Resources>
</Application>
```
... then use it in our View:
---
><font color="#63aebb" face="微软雅黑">... 然后在 View 中使用它：</font>

```xml
<UserControl x:Class="DesignMode.Views.SampleView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             mc:Ignorable="d"
             xmlns:vms="clr-namespace:DesignMode.ViewModels"
             d:DataContext="{Binding Source={StaticResource ViewModelLocator}, Path=SampleViewModel}">
   ...
</UserControl>
```


Enabling/Disabling Buttons at Design Time - 在设计时启用/禁用按钮
-----------------------------------------

In all of the examples above, we only bound the View's `DataContext`: we didn't do anything to its [View.ActionTarget](./Actions.md#the-viewactiontarget). This means that the enabledness of the button will not reflect the value of a guard property, if it exists - it will always be enabled.

**Note:** by default, Stylet will bind the `View.ActionTarget` to the corresponding ViewModel when it instantiates the View. However, at design time, Stylet is not responsible for instantiating the View, and so the `View.ActionTarget` isn't bound.

If you want the enabledless of the button to reflect its guard property, you need to add `s:View.ActionTarget="{Binding}"` to your View, e.g.

---
><font color="#63aebb" face="微软雅黑">在上面的所有例子中，我们只绑定了View 的 DataContext：我们没有对它的 [View.ActionTarget](./Actions.md#the-viewactiontarget) 做任何事情。这意味着按钮的启用不会反映保护属性的值（如果存在） - 它将始终启用。

>`注意:` 默认情况下，当 `View.ActionTarget` 实例化视图时，Stylet 会将它绑定到相应的 ViewModel。但是，在设计时，Stylet不负责实例化视图，所以 `View.ActionTarget` 不受约束。

>如果你希望按钮的启用能够反映其保护属性，则需要添加 `s:View.ActionTarget="{Binding}"` 到 View 中，例如：</font>

```xml
<UserControl x:Class="DesignMode.Views.SampleView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:s="https://github.com/canton7/Stylet"
             mc:Ignorable="d"
             xmlns:vms="clr-namespace:DesignMode.ViewModels"
             d:DataContext="{d:DesignInstance vms:SampleViewModel, IsDesignTimeCreatable=True}"
             s:View.ActionTarget="{Binding}">
   ...
</UserControl>
```

Using Substitute ViewModels - 使用替代的 ViewModel
---------------------------

Another solution to the problem of *"My ViewModel knows about design time, when it shouldn't"* is to have your View implement an interface (either a real interface, or a pretend one in your head), and you write another design-time-only View which implements the same interface, and contains dummy data. You then bind to it at design time in the same way: `d:DataContext="{d:DesignInstance vms:DummyViewModel, IsDesignTimeCreatable=True}"`. This is too much overhead for most developers, though.

WPF also has a concept of design time data, [see for example](http://blogs.msdn.com/b/wpfsldesigner/archive/2010/06/30/sample-data-in-the-wpf-and-silverlight-designer.aspx).

---
><font color="#63aebb" face="微软雅黑">解决“让 ViewModel 知道什么时候是设计时什么时候不是” 的解决方案是让 View 实现一个接口(一个实际的接口或一个模拟的接口)，然后编写一个设计时的视图，它实现相同的接口，并包含虚拟数据。然后在设计时以相同的方式绑定：`d:DataContext="{d:DesignInstance vms:DummyViewModel, IsDesignTimeCreatable=True}"`。不过，对于大多数开发人员而言，这耗费太多精力。

>WPF也有设计时数据的概念，[示例](http://blogs.msdn.com/b/wpfsldesigner/archive/2010/06/30/sample-data-in-the-wpf-and-silverlight-designer.aspx)。</font>

Why can't Stylet find my ViewModel automatically? - 为什么Stylet不能自动找到我的 ViewModel？
-------------------------------------------------

Since Stylet is capable of taking a ViewModel, and finding and instantiating its View, you may be tempted to ask why it can't do this the other way around: that is, at design time, automatically find the correct ViewModel for a given View, and instantiate it with the correct dependencies. There are a number of reasons why this is a Very Bad Idea:

1. We would need to add a means of translating a View name into a ViewModel name, which adds complexity to the `ViewManager` (particularly to anyone providing their own `ViewManager`).
2. We would need to instantiate an appropriate `IViewManager` implementation at design time. Since users can provide their own implementation, this would mean bring up the entire IoC container. Since this relies on the Assemblies property of the bootstrapper being set correctly, we would need to bring up the whole bootstrapper. This risks having nasty side effect (think starting services which do network commits, filesystem access, etc).
3. We would need to supply a ViewModel with all of its dependencies. This means instantiating services, which may have nasty side effects.
4. The ViewModel probably won't contain suitable dummy data anyway, so we haven't gained much.
5. Getting a Designer error (or worse, having network/filesystem side-effects) because we were trying to be too smart and caused some service to be started in the wrong way is painful to debug, and we shouldn't inflict that on people.

---
><font color="#63aebb" face="微软雅黑">由于 Stylet 能够获取 ViewModel，并查找和实例化 View，因此你可能会想问为什么它不能以相反的方式执行此操作：即，在设计时，自动为 View 查找正确的 ViewModel ，并使用正确的依赖项实例化它。这是一个非常糟糕的想法,因为有很多原因：

>1. 我们需要添加一种将 View 名称转换为 ViewModel 名称的方法，这会增加 `ViewManager` 的复杂性。

>2. 我们需要 `IViewManager` 在设计时实例化一个实现。由于用户可以提供自己的实现，这意味着需要调出 IoC 容器。由于这依赖于正确设置 bootstrapper 的 Assemblies 属性，因此我们需要调出整个 bootstrapper。这可能会产生令人讨厌的副作用（想想启动网络提交服务，文件系统访问等）。

>3. 我们需要为ViewModel提供其所有依赖项。这意味着实例化服务，这可能会产生令人讨厌的副作用。

>4. ViewModel 可能不会包含合适的虚拟数据，因此我们没有获得太多收益。

>5. 因为我们试图过于聪明而导致某些服务以错误的方式启动、导致设计器出错（或者更糟糕的是，出现网络/文件系统副作用），这对于调试来说是痛苦的，我们不应该造成这样的错误。</font>

s:View.Model and Embedded Views - s:View.Model 和 内嵌视图
-------------------------------

If you have an embedded View (e.g. `<ContentControl s:View.Model="{Binding SomeChildViewModel}"/>`) in your View, then Stylet will simply show a message along the lines of "View for ViewModelType.SomeChildViewModel", rather than trying to locate the correct View. This is for reasons very similar to why we don't locate ViewModels automatically: it would mean bringing up the IoC container (we'd need to find the user's `IViewManager` implementation), which means running the bootstrapper, which lets us do really dangerous stuff. Best avoided.

---
><font color="#63aebb" face="微软雅黑">如果你的 View 中有内嵌视图（例如`<ContentControl s:View.Model="{Binding SomeChildViewModel}"/>`），那么 Stylet 将只显示 “View for ViewModelType.SomeChildViewModel” 的消息，而不是尝试找到对应的视图。这与我们没有定位 ViewModel 的原因非常类似：它意味着启动 IoC 容器（我们需要找到用户的 `IViewManager` 实现）、运行 bootstrapper，这让我们做了非常危险的事情。最好避免。</font>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Logging - 日志](./Logging.md)