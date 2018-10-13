
The ViewModel-first approach is one that's crucial to Stylet's architecture, but unintuitive if you learnt MVVM in the tranditional View-first manner.

Hopefully this article will make everything clear.

The View-First Approach
-----------------------

Let's start by defining the View-first approach, and what exactly I mean by that. MVVM states that the ViewModel should know nothing about the View, but that View should be aware of the ViewModel. The obvious way to attach a View and ViewModel, then, is have the View construct its ViewModel in its codebehind - something like this:


<table><tr><td>C#</td><td>VB.NET</td>
<tr><td><pre lang="csharp">
public partial class MyView : Window
{
   public MyView()
   {
      InitializeComponent();
&nbsp;
      this.DataContext = new MyViewModel();
   }
}</pre>
</td><td><pre lang="vb.net">Partial Public Class MyView : Inherits Window
&nbsp;
  Public Sub New()
    InitializeComponent()
&nbsp;
    Me.DataContext = new MyViewModel()
&nbsp;
End Class</pre></td></tr></table>


This is fine. Views can create and own other views, meaning that you can compose your views into a hierarchy. All well and good.

The crunch comes when you've composed a couple of views, say something like this, where a shell contains a top bar and a frame, inside which any page can be shown:

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


The ViewModel-First Approach
----------------------------

The ViewModel-first approach accepts that a ViewModel shouldn't know anything about its View, but does not accept that the View should be responsible for constructing the ViewModel either. Instead, a third service is responsible for locating the correct View for a given ViewModel, and setting up its DataContext correctly.

The default implementation uses naming conventions to locate the correct View for a given ViewModel, replacing 'ViewModel' with 'View' in its name. That's explained in more detail in [[The ViewManager]].

This allows ViewModels to be created by other ViewModels. Which allows ViewModels to know about, and own, other ViewModels. Which allows you to compose your ViewModels properly.

There's another part of this trick, which is best explained by example:

&nbsp;
<table><tr><td>C#</td><td>VB.NET</td>
<tr><td><pre lang="csharp">
public class ShellViewModel
{
   public TopBarViewModel TopBar { get; private set; }
   // Stuff to instantiate and assign TopBarViewModel
}</pre>
</td><td><pre lang="vb.net">
Public Class ShellViewModel
&nbsp;
  Public Property TopBar as TopBarViewModel
  &#39; Stuff to instantiate and assign TopBarViewModel
End Class</pre></td></tr></table>


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


Delete the Code-Behind!
-----------------------

With this approach in place, there's nothing you actually need to do in the codebehind. You can of course do so, but there's very little you can't solve with [[Actions]] (for handling events), Converters, Attached Properties, and (most importantly) Attached Behaviors.

Stylet lets you delete the codebehind entirely (it will call `InitializeComponent` for you), and you are strongly encouraged to do so. Delete the codebehind!

*__Note__: If you're using VB.NET, sometimes your XAML namespaces will stop working if you delete the codebehind. If this is the case, simply recreate the codebehind with the matching filename, give it the correct namespace and class and then leave the remainder blank. For example, `MyView.xaml.vb` :*
```vb.net
Namespace Views
    Public Class MyView

    End Class
End Namespace
```
