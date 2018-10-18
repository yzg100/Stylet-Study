The bootstrapper is responsible for bootstrapping your application. It configures the IoC container, creates a new instance of your root ViewModel and displays it using the `WindowManager`. It also provides various other functions, described below.

The bootstrapper comes in two flavours: `BootstrapperBase<TRootViewModel>`, which requires you to configure the IoC container yourself, and `Bootstrapper<TRootViewModel>`, which uses Stylet's built-in IoC container, StyletIoC.

Example bootstrapper, using StyletIoC:

---
><font color="#63aebb" face="微软雅黑">Bootstrapper 负责引导你的应用程序。它配置 IoC 容器，创建 Root ViewModel 的新实例并使用 `WindowManager` 进行显示。它还提供了各种功能，如下。

>Bootstrapper 有两种形式: `BootstrapperBase<TRootViewModel>` 需要你自己配置 IoC 容器，`Bootstrapper<TRootViewModel>` 使用 Stylet 的内置 IoC 容器 StyletIoC。

>使用 StyletIoC 的 Bootstrapper 示例:</font>

```csharp
class Bootstrapper : Bootstrapper<MyRootViewModel>
{
   protected override void OnStart()
   {
        // This is called just after the application is started, but before the IoC container is set up.
        // 这是在应用程序启动之后但在设置IoC容器之前调用的。

        // Set up things like logging, etc
        //设置日志记录等内容
   }

   protected override void ConfigureIoC(IStyletIoCBuilder builder)
   {
        // Bind your own types. Concrete types are automatically self-bound.
        // 绑定自己的类型。具体类型自动绑定。

      builder.Bind&lt;IMyInterface&gt;().To&lt;MyType&gt;();
   }

   protected override void Configure()
   {
        // This is called after Stylet has created the IoC container, so this.Container exists, but before the
        // Root ViewModel is launched.
        // 这是在 Stylet 创建 IoC 容器之后调用的，所以 IoC container 存在，它在启动根视图模型之前。

        // Configure your services, etc, in here
        // 在此处配置你的服务等等
   }

   protected override void OnLaunch()
   {
        // This is called just after the root ViewModel has been launched
        // Root ViewModel启动后调用

        // Something like a version check that displays a dialog might be launched from here
        // 可以在这启动类似于显示对话框的版本检查
   }

   protected override void OnExit(ExitEventArgs e)
   {
        // Called on Application.Exit
        // 在 Application.Exit 上调用
   }

   protected override void OnUnhandledException(DispatcherUnhandledExceptionEventArgs e)
   {
        // Called on Application.DispatcherUnhandledException
        // 在 Application.DispatcherUnhandledException 上调用
   }
}
```

Using a Custom IoC Container - 使用自定义IoC容器
----------------------------

Using another IoC container with Stylet is easy. I've included bootstrappers for a number of popular IoC containers in the [Bootstrappers project](https://github.com/canton7/Stylet/tree/master/Bootstrappers). These are all unit-tested but not battle-tested: feel free to customize them.

Note that the Stylet nuget package / dll don't include these, as it would add unnecessary dependencies. Similarly, I don't publish IoC container-specific packages, as that's a waste of effort.

Copy the bootstrapper you want from the above link into your project somewhere. Then subclass it, as you would normally subclass `Bootstrapper<TRootViewModel>`, documented above. Then add your subclass to your App.xaml.cs, as documented in [[Quick Start]], e.g.

---
><font color="#63aebb" face="微软雅黑">在 Style 中使用 IoC 容器非常简单。在 [Bootstrappers project](https://github.com/canton7/Stylet/tree/master/Bootstrappers) 中包括了许多流行的 IoC 容器的 bootstrappers.这些都是经过单元测试但未经过实战测试：请任意定制它们。

>请注意，Stylet 的 nuget 包中不包含这些dll，因为它会增加不必要的依赖项。同样，我不发布 IoC 容器特定包，因为这是浪费精力事情。

>将你想要的 bootstrapper 从上面的链接复制到你的项目中使用它。正如你使用 `Bootstrapper<TRootViewModel>` 一样，在上面有文档说明。然后将子类添加到App.xaml中。如 [快速入门](./Quick-Start.md) 中一样。
</font>

```csharp
public class Bootstrapper : AutofacBootstrapper<ShellViewModel>
{
}
```

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

If you want to write your own bootstrapper for another IoC container, that's easy too. Take a look at the bootstrappers above to see what you need to do.

---
><font color="#63aebb" face="微软雅黑">如果你想为另一个 IoC 容器编写自己的引导程序，那也很容易。看看上面的 bootstrappers，看看你需要做什么。</font >

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[ViewModel First - 视图模型优先](./ViewModel-First.md)