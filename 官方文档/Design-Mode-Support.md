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

If your ViewModel would ordinarily only have a parameterless constructor, you can make use of [[Execute.InDesignMode|Execute: Dispatching to the UI thread]], like this:

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

```
d:DataContext="{d:DesignInstance vms:SampleViewModel, IsDesignTimeCreatable=True}"
```

Or, in full:

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

Intellisense and Dummy Data, using a ViewModelLocator
-----------------------------------------------------

The previous method has a huge drawback: it requires that your ViewModel knows about design mode, and contains code that is only ever invoked at design time. This is considered by some to be a huge code smell.

An alternative approach is to use a ViewModelLocator - a class, used only at design time, which can instantiate and configure your ViewModels. This means that any design-time-only local can go in the ViewModelLocator, and kept out of the Individual Views.

If this sounds complex, bear with me. It should all make sense in a minute.

First, let's grab ourselves a sample ViewModel:

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


Enabling/Disabling Buttons at Design Time
-----------------------------------------

In all of the examples above, we only bound the View's `DataContext`: we didn't do anything to its [[View.ActionTarget|Actions#the-viewactiontarget]]. This means that the enabledness of the button will not reflect the value of a guard property, if it exists - it will always be enabled.

**Note:** by default, Stylet will bind the `View.ActionTarget` to the corresponding ViewModel when it instantiates the View. However, at design time, Stylet is not responsible for instantiating the View, and so the `View.ActionTarget` isn't bound.

If you want the enabledless of the button to reflect its guard property, you need to add `s:View.ActionTarget="{Binding}"` to your View, e.g.

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

Using Substitute ViewModels
---------------------------

Another solution to the problem of *"My ViewModel knows about design time, when it shouldn't"* is to have your View implement an interface (either a real interface, or a pretend one in your head), and you write another design-time-only View which implements the same interface, and contains dummy data. You then bind to it at design time in the same way: `d:DataContext="{d:DesignInstance vms:DummyViewModel, IsDesignTimeCreatable=True}"`. This is too much overhead for most developers, though.

WPF also has a concept of design time data, [see for example](http://blogs.msdn.com/b/wpfsldesigner/archive/2010/06/30/sample-data-in-the-wpf-and-silverlight-designer.aspx).


Why can't Stylet find my ViewModel automatically?
-------------------------------------------------

Since Stylet is capable of taking a ViewModel, and finding and instantiating its View, you may be tempted to ask why it can't do this the other way around: that is, at design time, automatically find the correct ViewModel for a given View, and instantiate it with the correct dependencies. There are a number of reasons why this is a Very Bad Idea:

1. We would need to add a means of translating a View name into a ViewModel name, which adds complexity to the `ViewManager` (particularly to anyone providing their own `ViewManager`).
2. We would need to instantiate an appropriate `IViewManager` implementation at design time. Since users can provide their own implementation, this would mean bring up the entire IoC container. Since this relies on the Assemblies property of the bootstrapper being set correctly, we would need to bring up the whole bootstrapper. This risks having nasty side effect (think starting services which do network commits, filesystem access, etc).
3. We would need to supply a ViewModel with all of its dependencies. This means instantiating services, which may have nasty side effects.
4. The ViewModel probably won't contain suitable dummy data anyway, so we haven't gained much.
5. Getting a Designer error (or worse, having network/filesystem side-effects) because we were trying to be too smart and caused some service to be started in the wrong way is painful to debug, and we shouldn't inflict that on people.

s:View.Model and Embedded Views
-------------------------------

If you have an embedded View (e.g. `<ContentControl s:View.Model="{Binding SomeChildViewModel}"/>`) in your View, then Stylet will simply show a message along the lines of "View for ViewModelType.SomeChildViewModel", rather than trying to locate the correct View. This is for reasons very similar to why we don't locate ViewModels automatically: it would mean bringing up the IoC container (we'd need to find the user's `IViewManager` implementation), which means running the bootstrapper, which lets us do really dangerous stuff. Best avoided.

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Logging - 日志](./Logging.md)