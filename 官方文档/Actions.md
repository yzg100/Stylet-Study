
You have a button, and you want to click it and execute a method on your ViewModel? Actions cover this use-case.

---
><font color="#63aebb" face="微软雅黑">你有一个按钮，点击它并在 ViewModel 上执行一个方法? Action 覆盖了这个用例。</font>

Actions and Methods
-------------------

In 'traditional' WPF, you'd create a property on your ViewModel which implements the [ICommand](http://msdn.microsoft.com/en-us/library/system.windows.input.icommand%28v=vs.110%29.aspx) interface, and bind your button's Command attribute to it. This works fairly well (the ViewModel knows nothing about the View, and code-behind is not required), but it's a bit messy - you really want to be calling a method on your ViewModel, not executing a method on some property.

Stylet solves this by introducing Actions. Look at this:

---
><font color="#63aebb" face="微软雅黑">在 “传统” 的WPF中，在 ViewModel 上创建一个实现 [ICommand] (http://msdn.microsoft.com/en-us/library/system.windows.input.icommand%28v=vs.110%29.aspx) 接口的属性，并将按钮的 Command 属性绑定到该属性。这很好用（ViewModel 对 View 一无所知，并且不需要后台代码），但它有点乱 - 你真的想在 ViewModel 上调用一个方法，而不是在某些属性上执行方法。

>Stylet 通过引入 Action 来解决这个问题。如下：</font>

```csharp
class ViewModel
{
   public void DoSomething()
   {
      Debug.WriteLine("DoSomething called");
   }
}
```

```xml
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button Command="{s:Action DoSomething}">Click me</Button>
</UserControl>
```

As you might have guessed, clicking the button called the DoSomething method on the ViewModel to be called.

It's that simple.

If your method accepts a single argument, the value of the button's CommandParameter property will be passed. For example:

---
><font color="#63aebb" face="微软雅黑">你可能已经猜到了，单击按钮要调用的 ViewModel 上名为 DoSomething 的方法。

>就这么简单。

>如果你的方法接受单个参数，则将传递按钮的 CommandParameter 属性的值。例如：</font>

```csharp
class ViewModel
{
   public void DoSomething(string argument)
   {
      Debug.WriteLine(String.Format("Argument is {0}", argument));
   }
}
```

```xml
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button Command="{s:Action DoSomething}" CommandParameter="Hello">Click me</Button>
</UserControl>
```

Note that Actions also work on any ICommand property, on anything (e.g. a [KeyBinding](http://msdn.microsoft.com/en-us/library/system.windows.input.keybinding%28v=vs.110%29.aspx)).

请注意，Actions 也适用于任何 ICommand 属性（例如:[KeyBinding](http://msdn.microsoft.com/en-us/library/system.windows.input.keybinding%28v=vs.110%29.aspx)）。

Guard Properties - 保护属性
----------------

You can also control whether you button is enabled just as easily, using *Guard Properties*. A guard property for a given method is a boolean property which has the name "Can\<method name\>", so if your method is called "DoSomething", the corresponding guard property is called "CanDoSomething".

Stylet will check whether a guard property exists, and if so, will disable the button if it returns false, or enable it if it returns true. It will also watch for PropertyChanged notifications for that property, so you can change whether the button is enabled.

For example:

---
><font color="#63aebb" face="微软雅黑">你还可以使用 *保护属性* 轻松地控制按钮是否启用。给定方法的保护属性是一个 boolean 属性，其名称为 `Can<方法名称>`，因此如果你的方法被称为 `DoSomething`，相应的保护属性被称为 `CanDoSomething`。

>Stylet 将检查 保护属性是否存在，如果存在，且返回 false 则禁用按钮，如果返回 true 则启用按钮。它还会监视该属性的 PropertyChanged 通知，因此你可以更改按钮是否启用。

>示例：</font>

```csharp
class ViewModel
{
   public bool CanDoSomething
   {
      get { return this.someOtherConditionIsSatisfied(); }
   }
   
   public void DoSomething()
   {
      Debug.WriteLine(&quot;DoSomething called&quot;);
   }
}
```


Events
------

But what about if you want to call a ViewModel method when an event occurs? Actions have that covered as well. The syntax is exactly the same, although there's no concept of a guard property here.

```xml
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button Click="{s:Action DoSomething}">Click me</Button>
</UserControl>
```

The method which is called must have zero, one, or two parameters. The possible signatures are:

```csharp
public void HasNoArguments() { }

// This can accept EventArgs, or a subclass of EventArgs
public void HasOneSingleArgument(EventArgs e) { }

// Again, a subclass of EventArgs is OK
public void HasTwoArguments(object sender, EventArgs e) { }
```

The View.ActionTarget
---------------------

So far I've been telling a little white lie. I've been saying that the Action is invoked on the ViewModel, but that isn't strictly true. Let's go into a bit more detail.

Stylet defines an inherited attached property called View.ActionTarget. When a View is bound to its ViewModel, the View.ActionTarget on the root element in the View is bound to the ViewModel, and it's then inherited by each element in the View. When you invoke an action, it's invoked on the View.ActionTarget.

This means that, by default, actions are invoked on the ViewModel regardless of the current DataContext, which is probably what you want.

This is a very important point, and one that's worth stressing. The DataContext will probably change at multiple points throughout the visual tree. However, the View.ActionTarget will stay the same (unless you manually change it). This means the actions will always be handled by your ViewModel, and not by whatever object is being bound to, which is almost always what you want.

You can of course alter the View.ActionTarget for individual elements, for example:

```csharp
class InnerViewModel
{
   public void DoSomething() { }
}

class ViewModel
{
   public InnerViewModel InnerViewModel { get; private set; }
   public ViewModel()
   {
      this.InnerViewModel = new InnerViewModel();
   }
}
```


```xml
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button s:View.ActionTarget="{Binding InnerViewModel}" Command="{s:Action DoSomething}">Click me</Button>
</UserControl>
```

Here, InnerViewModel.DoSomething will be invoked when the button is clicked. Also, because the View.ActionTarget is inherited, any children of the Button will also have their View.ActionTarget set to the InnewViewModel.

Actions and Styles
------------------

Actions will not work from style setters. The classes required to do this in WPF are all internal, which means there is no way to fix the issue. You will need to use old-fashioned Commands in this (rare) case, unfortunately.


Gotchas - ContextMenu and Popup
-------------------------------

View.ActionTarget is of course an attached property, which is configured to be inherited by the children of whatever element it is set on. Like any attached property, and indeed the DataContext, there are certain boundaries it is not inherited across, such as:

 - Using a ContextMenu
 - Using a Popup
 - Using a Frame

In these cases, Stylet will do the best it can to find a suitable ActionTarget (it may, for example, find the the ActionTarget associated with the root element in the current XAML file), but this may not be exactly what you expect (e.g. it may ignore a `s:View.ActionTarget="{Binding ...}"` line you have somewhere in the middle of your page), or it may (in rare circumstances) fail to find an ActionTarget at all. 

In this case, please set `s:View.ActionTarget` to a suitable value. You may struggle to get a reference to anything outside of a ContextMenu from inside of one: I suggest the [BindingProxy](http://www.thomaslevesque.com/2011/03/21/wpf-how-to-bind-to-data-when-the-datacontext-is-not-inherited/) technique.

Additional Behaviour
--------------------

There are two cases which will stop an action from working properly: if the `View.ActionTarget` is null, or if the specified method on the `View.ActionTarget` doesn't exist. The default behaviour in each of these cases is as follows:

|         | View.ActionTarget == null | No method on View.ActionTarget 
|---------|---------------------------|-------------------------------
|Commands | Disable the control       | Throw an exception when the control is clicked
|Events   | Enable the control        | Throw an exception when the event is raised

The justification for this is that if the `View.ActionTarget` is null, you must have set it yourself, so you probably know what you're doing. However, if the specified method doesn't exist on the `View.ActionTarget`, that's probably a mistake, and you deserve to know.

Of course, this behaviour is configurable.

To control the behaviour when `View.ActionTarget` is null, set the `NullTarget` property on the `Action` markup extension so either `Enable`, `Disable`, or `Throw`. (Note that `Disable` is invalid when the Action is linked to an event, as we have no power to disable anything).

For example:

```xml
<Button Command="{s:Action MyMethod, NullTarget=Enable}"/>
<Button Click="{s:Action MyMethod, NullTarget=Throw}"/>
```

Similarly, you can set the `ActionNotFound` property to the same values:

```xml
<Button Command="{s:Action MyMethod, ActionNotFound=Disable}"/>
<Button Click="{s:Action MyMethod, ActionNotFound=Enable}"/>
```
[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[The WindowManager - 窗口管理器](./The-WindowManager.md)