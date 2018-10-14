
In a traditional View-first approach, if you want to display a new window or dialog, you create a new instance of the View, then call `.Show()` or `.ShowDialog()`.

In a ViewModel-first approach, you can't interact directly with the views, so you can't do this. The WindowManager solves this problem - calling `IWindowManager.ShowWindow(someViewModel)` will take that ViewModel, find its view, instantiate it, bind it to that ViewModel, and display it.

---
><font color="#63aebb" face="微软雅黑">在传统的 View-first 方式中，如果要显示新窗口或对话框，则创建View的新实例，然后调用 `.Show()` 或 `.ShowDialog()`。

>在 ViewModel-first 方式中，你无法直接与视图交互，因此你无法执行此操作。WindowManager 解决了这个问题 - 调用 `IWindowManager.ShowWindow(someViewModel)` 将获取 ViewModel，查找其 View 并实例化它，最后将其绑定到 ViewModel 并显示。</font>

&nbsp;
<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class SomeViewModel
{
   private IWindowManager windowManager;
   public SomeViewModel(IWindowManager windowManager)
   {
      this.windowManager = windowManager;
   }
&nbsp;
   public void ShowAWindow()
   {
      var viewModel = new OtherViewModel();
      this.windowManager.ShowWindow(viewModel);
   }
&nbsp;
   public void ShowADialog()
   {
      var viewModel = new OtherViewModel();
      bool? result = this.windowManager.ShowDialog(viewModel);
      // result holds the return value of Window.ShowDialog()
      if (result.GetValueOrDefault(true))
      {
         // DialogResult was set to true
      }
   }
}</pre>
</td><td valign="top"><pre lang="vb.net">
Class SomeViewModel
&nbsp;
    Private windowManager As IWindowManager
&nbsp;
    Public Sub New(ByVal windowManager As IWindowManager)
        Me.windowManager = windowManager
    End Sub
&nbsp;
    Public Sub ShowAWindow()
        Dim viewModel = New OtherViewModel()
        Me.windowManager.ShowWindow(viewModel)
    End Sub
&nbsp;
    Public Sub ShowADialog()
        Dim viewModel = New OtherViewModel()
        Dim result As Boolean? = Me.windowManager.ShowDialog(viewModel)
        &#39; Result holds the return value of Window.ShowDialog()
        If result.GetValueOrDefault(True) Then
        &#39; DialogResult was set to true
        End If
    End Sub
End Class</pre></td></tr></table>


Nice and easy! In addition, the introduction of the IWindowManager (rather than calling methods directly on the ViewModel) makes testing a lot easier.

To close a window or dialog from its ViewModel, use `Screen.RequestClose`, like this:

---
><font color="#63aebb" face="微软雅黑">很简单!此外，IWindowManager 的引入(而不是直接在 ViewModel 上调用方法)使测试更加容易。

>要从ViewModel关闭窗口或对话框，请使用 `Screen.RequestClose`，如下：</font>

&nbsp;
<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class ViewModelDisplayedAsWindow
{
   // Called by pressing the &#39;close&#39; button
   // 按 “关闭” 按钮调用 
   public void Close()
   {
      this.RequestClose();
   }
}
&nbsp;
class ViewModelDisplayedAsDialog
{
   // Called by pressing the &#39;OK&#39; button
   // 按 “确定”按钮调用 
   public void CloseWithSuccess()
   {
      this.RequestClose(true);
   }
}</pre>
</td><td valign="top"><pre lang="vb.net">
Class ViewModelDisplayedAsWindow
&nbsp;
    &#39; Called by pressing the  &#39;close&#39; button
    Public Sub Close()
        Me.RequestClose()
    End Sub
&nbsp;
  End Class
&nbsp;
Class ViewModelDisplayedAsDialog
&nbsp;
    &#39; Called by pressing the &#39;OK&#39; button
    Public Sub CloseWithSuccess()
        Me.RequestClose(True)
    End Sub
End Class</pre></td></tr></table>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[MessageBox - 消息框](./MessageBox.md)