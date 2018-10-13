
Want to get up and running as quickly as possible? This is the right place!

想要尽快启动并运行？就是这！
Automatic Option - 自动选择
----------------

If you're new to Stylet (and you're running VS2015 or later), this is the easiest way to get started.

1. Open Visual Studio, and create a new `WPF Application` project.
2. Open NuGet (right-click your project -> Manage NuGet Packages), and install the `Stylet.Start` package.

This will give you a working skeleton project.

When it has finished installing, uninstall Stylet.Start. 

**Note**: This will not work in VS2013 or earlier, due to NuGet weirdness. Follow the "Manual Option" section below.

Happy coding!

---
><font color="#63aebb" face="微软雅黑">如果你是 Stylet 新手（并且你正在运行VS2015或更高版本），这是最简单的入门方式。

>1、打开Visual Studio，然后创建一个新 WPF Application 项目。

>2、打开 NuGet（右键单击你的项目 - >管理NuGet包），然后安装 Stylet.Start 包。
这将为你提供一个有效的骨架项目。

>安装完成后卸载 Stylet.Start。

>注意：由于 Nuget 原因在 VS 2013 或更早版本中不起作用，所以请遵照下面的“手动选项”部分。

>编码快乐！</font>

Manual Option - 手动选择
-------------

If you don't want to use the `Stylet.Start` package and would prefer to create your own skeleton project, follow the instructions in this section.

1. Open Visual Studio, and create a new `WPF Application` project.
2. Open NuGet (right-click your project -> Manage NuGet Packages), and install the `Stylet.Start` package.

First off, delete `MainWindow.xaml` and `MainWindow.xaml.cs/vb`. You won't be needing them.

Next, you'll need a root View and a ViewModel. The View has to be a `Window`, but there are no other restrictions.

---
><font color="#63aebb" face="微软雅黑">如果你不想使用 `Stylet.Start` 包，并且希望创建自己的框架项目，请按照本节中的说明进行操作。 

>1、打开Visual Studio，然后创建一个新WPF Application项目。

>2、打开NuGet（右键单击你的项目 - >管理NuGet包），然后安装Stylet.Start包。

>首先，删除 `MainWindow.xaml` 和 `MainWindow.xaml.cs/vb`。你不需要它们。

>接下来，你将需要一个 Root View 和一个ViewModel。View 必须是 `Window`，除此之外没有其他限制。
</font>

```xml
<Window x:Class="Stylet.Samples.Hello.RootView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Height="300" Width="300">
    <TextBlock>Hello, World</TextBlock>
</Window>
```

The ViewModel can be any old class (for now - you might want it to be a [Screen or Conductor](https://github.com/canton7/Stylet/wiki/Screens-and-Conductors)).

---
><font color="#63aebb" face="微软雅黑">ViewModel 可以是任何以前的类(目前，你可能希望它是 [`Screen or Conductor`](./Screens-and-Conductors.md))。</font>
&nbsp;
<table><tr><td>C#</td><td>VB.NET</td>
<tr><td><pre lang="csharp">
public class RootViewModel
{
}</pre>
</td><td><pre lang="vb.net">
Public Class RootViewModel
&nbsp;
End Class</pre></td></tr></table>


Next, you'll need a bootstrapper. For now, you don't need anything special - just something to identify your root ViewModel. Later, you'll be able to configure your IoC container here, as well as other application-level stuff.

---
><font color="#63aebb" face="微软雅黑">接下来，你需要一个 bootstrapper。现在，你不需要任何特殊的东西 - 只需要识别你的根 ViewModel。稍后，你将能够在此处配置 IoC 容器以及其他应用程序级别的内容。</font>

&nbsp;
<table><tr><td>C#</td><td>VB.NET</td>
<tr><td><pre lang="csharp">
public class Bootstrapper : Bootstrapper&lt;RootViewModel&gt;
{
}</pre></td><td><pre lang="vb.net">
Public Class Bootstrapper Inherits Bootstrapper(Of RootViewModel)
&nbsp;
End Class</pre></td></tr></table>


Finally, this needs to be referenced as a resource in your `App.xaml`. You'll need to remove the `StartUri` attribute, and add `xmlns` entries for Stylet and your own application. Finally, you'll need to add Stylet's `ApplicationLoader` to the resources, and identify the bootstrapper you created above.

It should look something like this:
---
><font color="#63aebb" face="微软雅黑">最后，修改你的 `App.xaml` 文件。需要删除 `StartUri` 属性，并为 stylet 和你自己的应用程序添加 `xmln` 项。你需要将 Stylet 添加到 `ApplicationLoader` 资源中，并标识你在上面创建的 bootstrapper。

>它是这样的:</font>

```xml
<Application x:Class="Stylet.Samples.Hello.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet"
             xmlns:local="clr-namespace:Stylet.Samples.Hello">
    <Application.Resources>
       <s:ApplicationLoader>
            <s:ApplicationLoader.Bootstrapper>
                <local:Bootstrapper/>
            </s:ApplicationLoader.Bootstrapper>
        </s:ApplicationLoader>
    </Application.Resources>
</Application>
```

That's it! Run that, and you should get a window with 'Hello World' in it.

---
><font color="#63aebb" face="微软雅黑">就是这样!运行它，你会得到一个带有 “Hello World” 的窗口。</font>

The ApplicationLoader
---------------------

It's worth noting that `<s:ApplicationLoader>` above is a ResourceDictionary subclass.
This allows it to load in Stylet's built-in resources (see [[Screens and Conductors]]). You can choose not to load Stylet's resources like this:

---
><font color="#63aebb" face="微软雅黑">值得注意的是，<s:ApplicationLoader> 是 ResourceDictionary 的子类。这允许它加载 Stylet 的内置资源(参见[`Screen or Conductor`](./Screens-and-Conductors.md))。你可以选择不加载 Stylet 的资源，如下所示：</font>

```xml
<s:ApplicationLoader LoadStyletResources="False">
   ...
</s:ApplicationLoader>
```

If you want to add your own Resources / ResourceDictionaries to the Application, the simplest way is like this:

---
><font color="#63aebb" face="微软雅黑">如果要向应用程序添加自己的 Resources / ResourceDictionaries，最简单的方法如下： </font>

```xml
<Application.Resources>
    <s:ApplicationLoader>
        <s:ApplicationLoader.Bootstrapper>
            <local:Bootstrapper/>
        </s:ApplicationLoader.Bootstrapper>

        <Style x:Key="MyResourceKey">
            ...
        </Style>

        <s:ApplicationLoader.MergedDictionaries>
            <ResourceDictionary Source="MyResourceDictionary.xaml"/>
        </s:ApplicationLoader.MergedDictionaries>
    </s:ApplicationLoader>
</Application.Resources>
```

If this makes you uncomfortable for some reason, you can also do this:

---
><font color="#63aebb" face="微软雅黑">如果这让你感到不爽，你也可以这样做： </font>

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <s:ApplicationLoader>
                <s:ApplicationLoader.Bootstrapper>
                    <local:Bootstrapper/>
                </s:ApplicationLoader.Bootstrapper>
            </s:ApplicationLoader>

            <ResourceDictionary Source="MyResourceDictionary.xaml"/>
        </ResourceDictionary.MergedDictionaries>

        <Style x:Key="MyResourceKey">
            ...
        </Style>
    </ResourceDictionary>
</Application.Resources>
```
[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Bootstrapper - 引导程序](./Bootstrapper.md)