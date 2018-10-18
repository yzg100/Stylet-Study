
As we all know, WPF comes with its own MessageBox implementation - `System.Windows.MessageBox`. And that's fine, except that you can't call it from your ViewModel (well, you *can*, but it makes your ViewModel untestable). The usual workaround suggested online is "write your own".

Well, Stylet comes with its own MessageBox clone, which looks and behaves almost identically to the WPF one (including appearance, buttons, icons, auto-sizing, sounds, alignment, etc).

---
><font color="#63aebb" face="微软雅黑">我们都知道，WPF 有自己的 Messagebox 实现 -`System.Windows.MessageBox`。它很好，但你不能从你的 ViewModel 调用它。通常网上的建议是自己编写。

>Stylet 实现了自己的 MessageBox，它的外观和行为几乎与 WPF 一样(包括外观、按钮、图标、自动调整大小、声音、对齐等等)。</font>

Usage - 用法
-----

To use, simply call the `ShowMessageBox` method on `IWindowManager`, like this:

---
><font color="#63aebb" face="微软雅黑">只需在 `IWindowManager` 上调用 `ShowMessageBox` 方法，如下:</font>

```csharp
public MyViewModel
{
   private readonly IWindowManager windowManager;

   public MyViewModel(IWindowManager windowManager)
   {
      this.windowManager = windowManager;
   }

   public void ShowMessagebox()
   {
      var result = this.windowManager.ShowMessageBox("Hello");
   }
}
```

The MessageBox accepts all of the same options as as the WPF MessageBox, plus a few more.

---
><font color="#63aebb" face="微软雅黑">MessageBox 接受与 WPF MessageBox 相同的所有选项，以及其他一些选项。</font>


Customising the MessageBox - 自定义 MessageBox
--------------------------

Stylet's MessageBox is implemented as a ViewModel, `MessageBoxViewModel`, and its corresponding View, `MessageBoxView`. The ViewModel implements an interface, `IMessageBoxViewModel`, and the `ShowMessageBox()` method uses this interface to retrieve an instance of the ViewModel.

Therefore, you can provide you own custom implementation of `MessageBoxViewModel` and `MessageBoxView` by writing a ViewModel which implements `IMessageBoxViewModel`, and registering it with your IoC container. This will then be used by `ShowMessageBox()`.

If you just want to tweak the behaviour of the existing `MessageBoxViewModel`, you can. The following options are available:

---
><font color="#63aebb" face="微软雅黑">Stylet MessageBox 实现为 ViewModel - `MessageBoxViewModel`，和它对应的 View - `MessageBoxView`。ViewModel 实现接口 `IMessageBoxViewModel`，而 `ShowMessageBox()` 方法使用这个接口来检索 ViewModel 的实例。

>因此，你可以继承 `IMessageBoxViewModel` 接口，编写实现 `MessageBoxViewModel` 的 ViewModel，提供自己的 `MessageBoxViewModel` 和 `MessageBoxView` 的自定义实现，并将其注册到 IoC 容器中。然后使用 `ShowMessageBox()` 调用它。 

>如果你只想调整现有的 `MessageBoxViewModel` 的行为，可以使用以下选项：</font>

### Custom Button Text - 自定义按钮文本

You can edit the button text for any of the buttons on a per-application basis by modifying `MessageBoxViewModel.ButtonLabels`, which is a dictionary holding the text to display for each button. If you just want to edit the text for a particular MessageBox, `ShowMessageBox` will accept a dictionary allowing you to do just that:

---
><font color="#63aebb" face="微软雅黑">你可以编辑每个应用程序的任何按钮的文本 `MessageBoxViewModel.ButtonLabels`，这是一个包含每个按钮显示的文本的字典。如果你只想编辑特定 MessageBox 的文本，`ShowMessageBox` 接受一个允许你执行此操作的字典：</font>

```csharp
// Will display a MessageBox with the buttons "Yes please!" and "No, thanks"
// 将显示带有 "Yes please!" 和 "No, thanks" 按钮的消息框。
MessageBoxViewModel.ButtonLabels[MessageBoxResult.No] = "No, thanks";

this.windowManager.ShowMessageBox("Do you want breakfast?", 
    buttons: MessageBoxButton.YesNo, 
    buttonLabels: new Dictionary<MessageBoxResult, string>(){
        { MessageBoxResult.Yes, "Yes please!" },
});
```

### Custom Button Sets - 自定义按钮集

The `MessageBoxViewModel.ButtonToResults` dictionary specifies which buttons are shown for each `MessageBoxButton` enumeration value. Want to be able to display the buttons "OK", "Yes" and "No" together? Fiddle with this dictionary.

---
><font color="#63aebb" face="微软雅黑">`MessageBoxViewModel.ButtonToResults` 指定每个 `MessageBoxButton` 枚举值显示哪些按钮。设置该字典能够同时显示"OK"、"Yes" 和 "No" 按钮。</font>

### Custom Icons - 自定义图标

The `MessageBoxViewModel.IconMapping` dictionary specifies while icon is shown for which `MessageBoxImage` value. This dictionary must contain an entry for each `MessageBoxImage` value (note that different enum entries have the same value here), but a value may be null, in which case no icon is shown.

---
><font color="#63aebb" face="微软雅黑">`MessageBoxViewModel.IconMapping` 指定 `MessageBoxImage` 的图标。该词典必须包含每个 `MessageBoxImage` 值的条目(请注意，不同的枚举在这里具有相同的值)，值可以为 Null，为 Null 时不显示图标。</font>

### Custom Sounds - 自定义声音

`MessageBoxViewModel.SoundMapping` is a dictionary of which sound should be played for each `MessageBoxImage`. As with `IconMapping`, an entry must exist for each value in the `MessageBoxImage` enum, but null is a valid value (in which case no sound is played).

---
><font color="#63aebb" face="微软雅黑">`MessageBoxViewModel.SoundMapping` 是一个字典，每个 `MessageBoxImage` 都应该播放相应的声音，与 `IconMapping` 一样。可以设置为 Null。Null 时不播放声音。</font>

### Custom FlowDirection and TextAlignment

There are parameters to `IWindowManager.ShowMessageBox()` allowing you to specify the `FlowDirection` and `TextAlignment`.
If you do not specify these, then the defaults `MessageBoxViewModel.FlowDirection` and `MessageBoxViewModel.TextAlignment` are used.
You can change these defaults as well, if you wish.

---
><font color="#63aebb" face="微软雅黑">`IWindowManager.ShowMessageBox()` 允许你指定 `FlowDirection` 和 `TextAlignment`。如果未指定参数，则使用默认的 `MessageBoxViewModel.FlowDirection` 和 `MessageBoxViewModel.TextAlignment`。如果你愿意，也可以更改这些默认值。</font>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[The EventAggregator - 事件聚合器](./The-EventAggregator.md)