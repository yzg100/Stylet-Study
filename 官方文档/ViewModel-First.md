
The ViewModel-first approach is one that's crucial to Stylet's architecture, but unintuitive if you learnt MVVM in the tranditional View-first manner.

Hopefully this article will make everything clear.

---
><font color="#63aebb" face="微软雅黑">ViewModel-first方法对于 Stylet 体系结构至关重要，如果你以传统的 View-first 方式学习 MVVM，则不直观。

>希望这篇文章能弄清楚一切。</font>

The View-First Approach - 视图优先方法
-----------------------

Let's start by defining the View-first approach, and what exactly I mean by that. MVVM states that the ViewModel should know nothing about the View, but that View should be aware of the ViewModel. The obvious way to attach a View and ViewModel, then, is have the View construct its ViewModel in its codebehind - something like this:

---
><font color="#63aebb" face="微软雅黑">让我们首先定义View-first方法，MVVM指出，ViewModel 应该不知道 View，但是 View 应该知道 ViewModel。附加 View 和 ViewModel 的方法是让 View 在其代码后面构造它的 ViewModel - 像这样：</font>

```csharp
public partial class MyView : Window
{
   public MyView()
   {
      InitializeComponent();
      this.DataContext = new MyViewModel();
   }
}
```


This is fine. Views can create and own other views, meaning that you can compose your views into a hierarchy. All well and good.

The crunch comes when you've composed a couple of views, say something like this, where a shell contains a top bar and a frame, inside which any page can be shown:

---
><font color="#63aebb" face="微软雅黑">View 可以创建和拥有其他 View，你的 View 可以由多个 View 组合而成。 

>当你组合了几个 View 时，就会遇到困难。比如，一个 shell 包含一个顶部工具栏和一个框架，在其中可以显示任何页面： </font>

```xml
<!-- This is a window which contains a top bar and another page -->
<Window x:Class="MyNamespace.ShellView" ....>
   <StackPanel>
      <my:TopBarView/>
      <Frame x:Name="navigationFrame"/>
   </StackPanel>
</Window>
```

where `TopBarView` has its own ViewModel, `TopBarViewModel`. Fine.

Now say that `TopBarView` has a field containing some data you want to update, for example the title of the current page. Now, the `ShellViewModel` knows this (it's decided what the current page is, after all), but the `TopBarViewModel` doesn't (how would it? It doesn't really know about anything). The temptation is to expose a dependency property on the `TopBarView` and bind it into the `ShellViewModel`, like so:

---
><font color="#63aebb" face="微软雅黑">其中 `TopBarView` 有自己的 ViewModel `TopBarViewModel`。

>现在假设 `TopBarView` 有你要更新的数据的字段，例如当前页面的标题。`ShellViewModel` 知道这一点，但是 `TopBarViewModel` 没有。将依赖属性暴露在 `TopBarView` 上，并将其绑定到 `ShellViewModel` 中，如下：</font>

```xml
<Window x:Class="MyNamespace.ShellView" .... x:Name="rootObject">
   <StackPanel>
      <my:TopBarView CurrentPageTitle="{Binding CurrentPageTitle, ElementName=rootObject}"/>
      <Frame x:Name="navigationFrame"/>
   </StackPanel>
</Window>
```

but that's just nasty. You've now got one-and-a-bit views bound to the `ShellViewModel`.

Another major concern is displaying windows and dialogs. In traditional MVVM, this is a bit of a pain. One option is to instantiate and display the View (using `Show()` or `ShowDialog()`) from inside the ViewModel (which makes it, or at least that bit of it, untestable). The better option is to instantiate the View you want to display in the codebehind of your View, and show it from there. This means you need to establish ways of telling to View to display this dialog, and a way of getting the results of the dialog back to the ViewModel. 

Indeed, setting the content for the `Frame` above requires instantiating a View to put in it. This has the same dilemma - either the ViewModel instantiates it (making it untestable), or the View does (leading to communication pain).

Either way, this approach has some nastiness.

---
><font color="#63aebb" face="微软雅黑">这非常糟糕。你现在已经有一个和 View 绑定的 `ShellViewModel`。

>另一个是显示窗口和对话框。在传统的 MVVM 中，这有点麻烦。一种方法是从 ViewModel 内部实例化并显示 View (使用 `Show()` 或 `ShowDialog()`)(使其成为或至少其中的一部分，是不可测试的)。更好的选择是实例化你想要显示在 View 代码中的 View，并在那里显示它。这意味着你需要建立一种方法来告诉 View 显示这个对话框，以及一种将对话框的结果返回到 ViewModel 的方法。

>实际上，假设上述内容 `Frame` 需要实例化一个 View 放入其中。这有两个相同的困境 - 要么 ViewModel 实例化它（使其不可测试），要么 View 实现（导致沟通麻烦）。

>不管是哪种方式，都有一些不好的地方。 </font>

The ViewModel-First Approach - 视力模型优先方法
----------------------------

The ViewModel-first approach accepts that a ViewModel shouldn't know anything about its View, but does not accept that the View should be responsible for constructing the ViewModel either. Instead, a third service is responsible for locating the correct View for a given ViewModel, and setting up its DataContext correctly.

The default implementation uses naming conventions to locate the correct View for a given ViewModel, replacing 'ViewModel' with 'View' in its name. That's explained in more detail in [[The ViewManager]].

This allows ViewModels to be created by other ViewModels. Which allows ViewModels to know about, and own, other ViewModels. Which allows you to compose your ViewModels properly.

There's another part of this trick, which is best explained by example:

---
><font color="#63aebb" face="微软雅黑">ViewModel-first 方法接受 ViewModel 不知道其 View 的任何内容，不接受 View 应该负责构造ViewModel。相反，另一个服务负责为给定的 ViewModel 定位正确的 View，并正确设置 DataContext。

>默认实现使用命名约定来查找给定 ViewModel 的正确视图，将 “ViewModel” 替换为其名称中的 “View”。这在[ViewManager](./The-ViewManager.md)中有更详细的解释。

>允许 ViewModels 由其他 ViewModels 创建。允许 ViewModels 了解并拥有其他 ViewModels。这让你可以正确编写 ViewModel。

>这是技巧的另一部分，用例子解释：</font>

```csharp
public class ShellViewModel
{
   public TopBarViewModel TopBar { get; private set; }
   // Stuff to instantiate and assign TopBarViewModel
   // 实例化和分配 TopBarViewModel
}
```


```xml
<Window x:Class="MyNamespace.ShellView"
        xmlns:s="https://github.com/canton7/Stylet" .....>
   <StackPanel>
      <ContentControl s:View.Model="{Binding TopBar}"/>
      <!-- ... -->
   </StackPanel>
</Window>
```

The `View.Model` attached property will fetch the ViewModel it's bound to (in this case it's an instance of `TopBarViewModel`), and locate the correct view (`TopBarView`). It will instantiate an instance, and set it as the content of that `ContentControl`.

The upshot is that the `TopBarView` can get the name of the current page from its `TopBarViewModel`, and the `TopBarViewModel` can be told this by the `ShellViewModel`. Problem solved!

The `ContentControl` trick also works well for navigation:

---
><font color="#63aebb" face="微软雅黑">`View.Model` 附加属性将获取其绑定到的 ViewModel `TopBarViewModel`，并找到正确的 View（`TopBarView`）。它将实例化一个实例，并将其设置为 `ContentControl` 的内容 。

>其结果是，`TopBarView` 可以从 `TopBarViewModel` 中获得当前页面的名称，而 `TopBarViewModel` 则可以通过 `ShellViewModel` 来告诉它。问题解决！

>`ContentControl` 技巧也适用于导航：</font>



```xml
<Window x:Class="MyNamespace.ShellView"
        xmlns:s="https://github.com/canton7/Stylet" .....>
   <StackPanel>
      <ContentControl s:View.Model="{Binding TopBar}"/>
      <ContentControl s:View.Model="{Binding CurrentPage}"/>
   </StackPanel>
</Window>
```

The `ShellViewModel` will then navigate to a new page by instantiating a new instance of that page's `ViewModel`, and assigning it to the property `CurrentPage`. Note how the `ShellViewModel` no longer needs to know anything about views. *It hasn't had to instantiate a single view*. This is a very important, useful, and powerful point.

Dialogs and Windows are handled in much the same way by [[The WindowManager]]. This takes a given ViewModel instance, and displays its View as a dialog or window.

---
><font color="#63aebb" face="微软雅黑">然后，`ShellViewModel` 将通过实例化该页面的新实例 ViewModel 并将其分配给 `CurrentPage` 属性来导航到新页面。请注意 `ShellViewModel` 不再需要了解有关视图的任何信息。它不必实例化单个视图。这是一个非常重要、有用和强大的要点。

>[WindowManager](./The-WindowManager.md) 以大致相同的方式处理 Dialog 和 Windows。它接收给定的 ViewModel 实例，并将其视图显示为对话框或窗口。</font>

Delete the Code-Behind! - 删除后台代码!
-----------------------

With this approach in place, there's nothing you actually need to do in the codebehind. You can of course do so, but there's very little you can't solve with [[Actions]] (for handling events), Converters, Attached Properties, and (most importantly) Attached Behaviors.

Stylet lets you delete the codebehind entirely (it will call `InitializeComponent` for you), and you are strongly encouraged to do so. Delete the codebehind!

*__Note__: If you're using VB.NET, sometimes your XAML namespaces will stop working if you delete the codebehind. If this is the case, simply recreate the codebehind with the matching filename, give it the correct namespace and class and then leave the remainder blank. For example, `MyView.xaml.vb` :*

---
><font color="#63aebb" face="微软雅黑">有了这种方法，你真的不需要做任何事情。当然，你也可以这样做，但是你很难用 [Actions](./Actions)（处理事件），转换器，附加属性和（最重要的）附加行为来解决。

>Stylet 允许你完全删除后台代码(它会为你调用 `InitializeComponent`)，强烈建议你这样做，删除后台代码!

>注意：如果你使用的是 VB.NET，删除后台代码有时会使你的 XAML 命名空间将停止工作。如果是这种情况，只需使用匹配的文件名重新创建后台代码，为其提供正确的命名空间和类，然后将其余部分留空。例如，`MyView.xaml.vb`：</font>

```vb.net
Namespace Views
    Public Class MyView

    End Class
End Namespace
```
[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Actions - 活动](./Actions.md)