On every project ever I've needed to debug a binding. The easiest way of doing this is to put a converter on the binding, which does nothing except log the values it sees. `DebugConverter` is an implementation of such a converter, and will log every call to Visual Studio's Output window, provided you're running a debug build.

Basic usage is very easy:

---
><font color="#63aebb" face="微软雅黑">在每个项目上我都需要调试绑定。最简单的方法是在绑定上放置一个转换器，除了记录它看到的值之外什么都不做。`DebugConverter` 就是一个这样的转换器的实现，它将记录每次调用，并在Visual Studio 的输出窗口，前提是你正在运行调试版本。

>用法很简单：</font>

```xml
<TextBox Text="{Binding MyProperty, Converter={x:Static s:DebugConverter.Instance}}"/>
```

If you want to have several instances active at once, and want to give each one a name (which is included in its output), you can do this:

---
><font color="#63aebb" face="微软雅黑">如果你希望一次激活多个实例，并希望为每个实例指定一个名称（包含在其输出中），可以执行以下操作：</font>

```xml
<!-- In any .Resources section - doesn't have to be Window.Resources -->

<!--在任何.Resources部分 - 不必是Window.Resources-->
<Window.Resources>
   <s:DebugConverter x:key="debugConverter" Name="MySpecialName"/>
</Window.Resources>
<!-- Later in code -->
<TextBlock Text="{Binding MyProperty, Converter={StaticResource debugConverter}}"/>
```

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[BoolToVisibilityConverter](./BoolToVisibilityConverter.md)