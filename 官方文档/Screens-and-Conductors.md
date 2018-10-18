
Screens and Conductors are a simple topic, but one which requires a few mental leaps, and requires you to cover all parts of them before they'll make sense. Trust me, it's well worth your time to read this article - they're hugely powerful, and well worth the time investment.

---
><font color="#63aebb" face="微软雅黑">Screens 和 Conductors 是一个简单的话题，但它需要一些思维上的跳跃，并且要求你在理解它们之前必须涵盖它们的所有部分。相信我，这是非常值得你花时间阅读这篇文章 - 他们非常强大，非常值得投入时间。</font>

ViewModel Lifecycles - ViewModel 生命周期
--------------------

A good place to start at is by looking at ViewModel lifecycles.

Imagine a tabbed interface - something like Visual Studio, which has (very simplistically) a shell (containing the menus, toolbars, etc) and a TabControl containing your editor tabs. In Stylet, each of those editor tabs would be backed by its own ViewModel.

Now, one of those ViewModels will start its life by being instantiated. Next up, it will be displayed. After that, it might be shown or hidden depending on which tab is currently active, before finally being closed. Just before it's closed it has the opportunity to prevent that closure to prompt you to save the file.

This, in a nutshell, is a ViewModel's lifecycle: it's created, then Activated (displayed to the user). After that, it can be Deactivated (still alive but not shown) and Activated again any number of times, before finally being Closed (after being asked whether it's ready to be closed).

---
><font color="#63aebb" face="微软雅黑">一个好的开始是查看 ViewModel 生命周期。

>想象一个选项卡式界面 - 类似 Visual Studio，它（非常简单）一个 shell（包含菜单，工具栏等）和一个包含编辑器选项卡的 TabControl。在 Stylet 中，每个编辑器选项卡都将由其自己的 ViewModel 支持。

>现在，其中一个 ViewModel 将通过实例化开始其生命周期。接下来，它将显示。之后，可能会显示或隐藏，具体取决于最终关闭之前当前处于活动状态的选项卡。就在它关闭之前，它有机会阻止关闭并提示你保存文件。

>简而言之，这是一个 ViewModel 的生命周期：它被创建，然后被激活（显示给用户）。之后，它可以被取消激活（未被销毁只是未显示）并且在最终被关闭之前（在被询问是否准备好关闭之后）再次激活任何次数。</font>

IDisposable
-----------

It's worth noting that if a ViewModel implements `IDisposable`, then it will be disposed after it's closed by its parent (unless the parent's `DisposeChildren` property is false).

---
><font color="#63aebb" face="微软雅黑">值得注意的是，如果 ViewModel 实现了 `IDisposable`，那么它将在它的父节点关闭之后被释放（除非父节点的 `DisposeChildren` 属性为 false）。</font>

Introducing Conductors - Conductors 介绍
----------------------

Now, a ViewModel doesn't magically know when it's been shown, hidden, or closed. It has to be told. This is the role of a Conductor.

A Conductor is, simply, a ViewModel which owns another ViewModel, and knows how to manage its lifecycle.

In our Visual Studio example, the Conductor is going to be the ViewModel which owns the TabControl inside which the editor ViewModels are displayed, so probably the Shell ViewModel. Whenever the user selects a  new editor tab, the Conductor is going to Deactivate the old tab, and Activate the new tab. When the user closes a tab, the Conductor is going to tell that tab that it's been Closed, then decide which is the next tab to display, and Activate that.

And that's it, really. ViewModels have a lifecycle, and that's implemented by the Conductor which owns the ViewModel.

This has been quite abstract so far - let's go into the details.

---
><font color="#63aebb" face="微软雅黑">现在，ViewModel 不会神奇地知道它何时被显示，隐藏或关闭。必须要告诉它。这是 Conductor 的作用。

>简单地说，Conductor 是一个拥有另一个 ViewModel 的 ViewModel，并且知道如何管理它的生命周期。

>在 Visual Studio 示例中，Conductor 将成为 ViewModel，它拥有 TabControl，其中有显示了编辑器的 ViewModel，因此可能是 ShellViewModel。每当用户选择新的编辑器选项卡时，Conductor 将停用旧选项卡，并激活新选项卡。当用户关闭选项卡时，Conductor 将告诉该选项卡它已被关闭，然后决定下一个要显示的选项卡，并激活它。

>就是这样，ViewModels 有生命周期，由拥有 ViewModel 的 Conductor 实现。

>以上都是非常抽象的 - 让我们看看细节。</font>

IScreen and Screen
------------------

As we saw above, a ViewModel's lifecycle is managed by a Conductor calling methods on that ViewModel. Those methods are defined in a set of independent interfaces - if you implement the interface, and that ViewModel's managed be a Conductor, that method will get called. You can pick and choose which interfaces you want, if you want.

There's an overarching interface called `IScreen` which composes them all, and a default implementation called `Screen`. This behaves really very well, and you'll probably never need to implement your own - but you can if you want.

 - `IScreenState`: Used to activate, deactivate, and close the ViewModel. Has `Activate`, `Deactivate`, and `Close` methods method, as well as events and properties to track changes in the screen's state.
 - `IGuardClose`: Used to ask the ViewModel whether it can close. Has a `CanCloseAsync` method.
 - `IViewAware`: Sometimes a ViewModel needs to know about its View (when it's attached, what it is, etc). This interface allows that through a `View` property and an `AttachView` method.
 - `IHaveDisplayName`: Has a `DisplayName` property. This name is used as the title for windows and dialogs displayed using [[The WindowManager]], and is also useful for things like TabControls.
 - `IChild`: It can be advantageous for a ViewModel to know what Conductor is managing it (to request that it be closed, for example). If the ViewModel implements `IChild`, it will be told this.

---
><font color="#63aebb" face="微软雅黑">如上所述，ViewModel 的生命周期由 ViewModel 上的 Conductor 调用方法管理。这些方法是在一组独立接口中定义的 - 如果你实现了该接口，并且 ViewModel 托管的是 Conductor，那么该方法将被调用。如果你愿意，你可以选择你想要的接口。

>有一个名为 `IScreen` 的总体接口，它构成了所有的界面，还有一个默认的实现，称为 `Screen`。它的性能非常好，你可能永远不需要实现自己的接口 - 但如果你愿意，你可以这样做。 

>- `IScreenState`：激活、禁用和关闭 ViewModel。拥有 `Activate`, `Deactivate`,和 `Close` 方法，以及跟踪 screen 状态变化的事件和属性。 

>- `IGuardClose`：询问 ViewModel 是否可以关闭。拥有 `CanCloseAsync` 方法。

>- `IViewAware`：有时候 ViewModel 需要知道它的 View(当它被附加时，它是什么，等等)。这个接口通过一个 `View` 属性和一个 `AttachView` 方法来实现。

>- `IHaveDisplayName`：是否具有 `DisplayName` 属性。这个名称可作为 [WindowManager](./The-WindowManager.md) 显示的窗口和对话框的标题，对于 TabControl 控件也很有用。

>- `IChild`：对于一个 ViewModel 来说，知道是什么 Conductor 在管理它是有用的(例如，要求关闭它)。如果 ViewModel 实现了 `IChild`，就会接收到通知。 
</font>

Note that there's no guarantee of the order in which Activate, Deactivate, and Close are called in - a ViewModel can be activated twice in a row, then closed without being deactivated. It's up to the ViewModel to notice these things, and react accordingly. Stylet's `Screen` does this.

`Screen` has some virtual methods you're encouraged to override if you want:

 - `OnInitialActivate`: Called the first time that the Screen is activated, and never again. Useful for setting up things which you don't want to set up in the constructor.
 - `OnActivate`: Called when the Screen's activated. Will only be called if the Screen wasn't already active.
 - `OnDeactivate`: Called when the Screen's deactivated. Will only be called if the Screen wasn't already deactivated.
 - `OnClose`: Called when the Screen's closed. Will only be called once. Will only be called when the Screen's deactivated.
 - `OnViewLoaded`: Called when the View's `Loaded` event is fired.
 - `CanCloseAsync`: Called when the Conductor wants to know whether the Screen can close. By default, retuns `Task.FromResult(this.CanClose)`, but you can add your own asynchronous logic here.
 - `CanClose`: Called by `CanCloseAsync` by default. This is just a convenience. If you want to decide whether or not you can close synchronously, override `CanClose`. If you want to decide asynchronously, override `CanCloseAsync`.
 - `RequestClose(bool? dialogResult = null)`: You call this when you want to request a closure from your owning Conductor. The DialogResult parameter is used if you're being shown in a dialog.

Screen derives from [[PropertyChangedBase]], so it's easy to raise PropertyChanged notifications.

You'll probably find that all of your ViewModels are subclasses of Screen. That's not to say that they need to be - you can create your own implementation of `IScreen`, or pick and choose the interfaces you want to implement from above - but it's convenient and powerful.

---
><font color="#63aebb" face="微软雅黑">注意，不能保证调用Activate、Deactivate 和 Close 的顺序 - ViewModel 可以被连续激活两次，然后在不被激活的情况下关闭。这取决于 ViewModel 来注意这些事情，并做出相应的反应。Stylet 的 `Screen` 是这样做的。

>`Screen` 有一些虚拟方法，你可以覆盖它们：

>- `OnInitialActivate`：仅在 Screen 第一次激活时调用。用于设置你不想在构造函数中设置的内容。

>- `OnActivate`：当 Screen 激活时调用。只有在 Screen 尚未激活的情况下才会调用。

>- `OnDeactivate`： Screen 停用时调用。只在 Screen 尚未停用时才会调用。

>- `OnClose`： Screen 关闭时调用。只会被调用一次。只在 Screen 停用时调用。

>- `OnViewLoaded`：在触发 View 的 `Loaded` 事件时调用。

>- `CanCloseAsync`：当 Conductor 想知道 Screen 是否可以关闭时调用。默认情况下，返回`Task.FromResult(this.CanClose)`，你可以在此处添加自己的异步处理逻辑。

>- `CanClose`：为了方便，`CanCloseAsync` 只是默认情况下调用。如果你想决定是否可以同步关闭，请重写 `CanClose`。如果要异步决定，则重写 `CanCloseAsync`。 

>- `RequestClose(bool? dialogResult = null)`：当你向 Conductor 请求关闭时，可以调用这个函数。如果在对话框中，则使用对话框 DialogResult 参数。

 >Screen 派生自 [PropertyChangedBase](./PropertyChangedBase.md)，因此很容易引发 PropertyChanged 通知。

 >你可能会发现，所有的 ViewModel 都是 Screen 的子类。这并不是必要的 - 你可以创建自己的 `IScreen` 实现，或者从上面选择你想要实现的接口，它非常方便且强大。 </font>

Conductors in Detail - Conductors 细节
-------------------

Conductors come in a variety of flavours, each of which has its own use-cases. A conductor can own a single ViewModel (think navigation with one page shown at a time), or multiple ViewModels. Those with multiple ViewModels can have have just one active at a time (think the TabControl from the Visual Studio example above), or all of them (think grid with lots of independent elements). Conductors can also add behaviour such as keeping a record of which ViewModels they've shown (useful for navigation). 

As with the `Screen` class, Stylet defines a number of interfaces which are of interest to conductors, and a number of implementations (depending on the sort of conductor behaviour you want), although you can of course implement your own.

The primary interface is `IConductor<T>`, which represents a conductor as you would interact with it. It has the following methods

 - `ActivateItem(T item)`: Take the given item, and activate it. Whether this deactivates a previous item is determined by the conductor.
 - `DeactivateItem(T item)`: Take the given item, and deactivate it. Whether this activates another item is determined by the conductor.
 - `CloseItem(T item)`: Take the given item, and close it. Whether this causes another item to be activated to take its place is conductor-specific.

The conductors which have a single active item (regardless of how many inactive items they may have) also implement `IHaveActiveItem<T>`, which has the single property `ActiveItem`.

All of the built-in conductors will set an item's `Parent` property to itself, if the item implements `IChild`. All of the built-in conductors additionally implement `IChildDelegate`, which allows the child to request that it be closed (by calling `CloseItem`). In the default `Screen` implementation, calling `Screen.RequestClose` will cause the screen to call `CloseItem` on its parent (provided its parent implement `IChildDelegate`), which in turn causes its parent (if it exists) to close it.

---
><font color="#63aebb" face="微软雅黑">Conductor 有多种不同的风格，每一种都有自己的用途。一个 Conductor 可以拥有一个 ViewModel (比如一次显示一个页面的导航)，或者多个 ViewModel。有多个 ViewModel 的 Conductor 一次只能有一个活动的 View (比如上面的 Visual Studio 示例中的 TabControl)，或者所有 ViewModel（想想有很多独立元素的网格）。Conductor 还可以添加行为，例如记录显示的 ViewModel（对导航很有用）。

>与 `Screen` 类一样，Style定义了许多 conductor 感兴趣的接口，以及一些实现(取决于你想要的 conductor 行为类型)，你当然可以实现自己的接口。 

>主接口是 `IConductor<T>`，它代表conductor，就像你将与它交互一样。它有以下方法：

>- `ActivateItem(T item)`：获取给定 item 并激活它。是否取消激活`上一个` item 由 conductor 决定。

>- `DeactivateItem(T item)`：获取给定 item 并停用它。是否激活`另一个` item 由 conductor 决定。

>- `CloseItem(T item)`：关闭指定的 item，是否导致`另一个` item 被激活并取代它由conductor 决定。

>具有单个活动项(不管它们可能具有多少非活动项)的 conductor 还实现了 `IHaveActiveItem<T>` 接口，它具有当前活动项属性 `ActiveItem`。

>如果 item 实现了 `IChild` 接口，所有内嵌 conductor 都将为 items 的 `Parent` 属性设置为自己。所有内嵌 conductor 都额外实现了 `IChildDelegate`，它允许 child 通过调用 `CloseItem` 请求关闭。默认的 `Screen` 调用 `Screen.RequestClose` 将导致 screen 调用父组件上的 `CloseItem` 方法(如果它的父组件实现了 `IChildDelegate`接口)，这反过来又会导致它的父组件(如果存在的话)关闭它。</font>

Built-In Conductors - 内嵌 Conductor
-------------------

Stylet comes with some conductors built-in, which perform in a number of intuitive ways.

All of these conductors derive from `Screen`, allowing conductors to easily own other conductors. This means you can compose your conductors and screens in any way you want.

---
><font color="#63aebb" face="微软雅黑">Style 附带了一些 内嵌 Conductor，它们以多种直观的方式执行。 

>所有这些 conductor 继承至 `Screen`，允许 conductor 容易地拥有其它 conductor。这意味着你可以以任何方式组合 conductor 和 screen。</font>

### `Conductor<T>`

This very simple conductor owns a single ViewModel (of type `T`), which is exposed as the `ActiveItem`. The `ActivateItem` method is used to replace the current `ActiveItem` with a new ViewModel instance, and will activate the new item and close the old one. Whenever the `Condcutor<T>` is activated, it activates its `ActiveItem`; likewise it deactivates and closes its `ActiveItem` when it is deactivated or closed, respectively.

When asked whether it can be closed (when `CanCloseAsync` is called), it returns whatever its `ActiveItem` returned, or true if there's no `ActiveItem`.

The `ActiveItem` can also be set directly, which has the same effect as calling `ActivateItem` on it.

A ViewModel for this Conductor will look something like this - a ContentControl bound to the conductor's ActiveItem:

---
><font color="#63aebb" face="微软雅黑">这个简单的 conductor 拥有一个ViewModel（类型 `T`），它被公开为 `ActiveItem`。`ActivateItem` 方法用于用新的 ViewModel 实例替换当前的 `ActiveItem`，激活新项并关闭旧项。当 `Condcutor<T>` 被激活时，它会激活它的 `ActiveItem`;同样，当 `ActiveItem` 被分别停用或关闭时，它也会分别停用和关闭。

>当被问及它是否可以被关闭时(当调用 `CanCloseAsync` 时)，它会返回 `ActiveItem` 的调用返回值，如果没有`ActiveItem` 则返回true。

>也可以直接设置 `ActiveItem`，其效果与调用 `ActivateItem` 相同。

>Conductor 的 ViewModel 看起来是这样 - 将 conductor.ActiveItem 绑定到 ContentControl：</font>

```xml
<Window x:Class="MyNamespace.ConductorViewModel"
        xmlns:s="https://github.com/canton7/Stylet" ....>
   <ContentControl s:View.Model="{Binding ActiveItem}"/>
</Window>
```


### `Conductor<T>.Collection.OneActive`

This conductor owns many items, but only one can be active at a time. In this way, it models the behaviour of a TabControl - many tabs can exist at once, but only one may be displayed at a time.

It owns a collection of `T`'s called `Items`, and one of these is further blessed with also being the `ActiveItem`. Calling `ActivateItem` will add the item passed to the `Items` collection, and will also activate it and set it as the `ActiveItem`; if the `ActiveItem` was previously set, its old value is deactivated, and remains in the `Items` collection.

Calling `DeactivateItem` or `CloseItem` on an item will cause that item to be deactivated and closed, respectively. Since it's no longer active, it can't remain as the `ActiveItem` - instead, another item is chosen to be the `ActiveItem`, and is activated and set as such. By default, the new `ActiveItem` is the one which exists in the `Items` collection just in front of the item being deactivated/closed.

The `Items` collection can be manipulated directly, if you want. The `ActiveItem` can also be set directly, which has the same effect as calling `ActivateItem` and passing that item.

A ViewModel with a TabControl using this conductor could look like this (see below for the short version):

---
><font color="#63aebb" face="微软雅黑">该 conductor 拥有多个 Item，但一次只能有一个 Item 被激活。通过这种方式，它可以模拟 TabControl 的行为 - 许多选项卡可以同时存在，但一次只能显示一个。

>它拥有一个类型为 `T` 的集合 `Items`，其中一个被赋予了 `ActiveItem`。调用 `ActivateItem` 将 item 添加到 `Items` 集合中，同时将其激活并设置为 `ActiveItem`；如果 `ActiveItem` 之前设置过，则被停用，并保留在 `Items` 集合中。 

>在一个 item 上调用 `DeactivateItem` 或 `CloseItem` 将分别导致该 item 被停用和关闭。它不再是被激活的，所以它不再是 `ActiveItem` - 另一个 item 会被激活并设置为 `ActiveItem`。新的 `ActiveItem` 是存在于 `Items` 集合中的一个，位于被停用/关闭的项前面。

>如果你愿意，可以直接操作 `Items` 集合。`ActiveItem` 也可以直接设置，其作用与调用`ActivateItem` 传递该 item 相同。

>使用此 conductor 的 TabControl ViewModel 可以如下(参见下面的简短版本):</font>

```xml
<TabControl ItemsSource="{Binding Items}" SelectedItem="{Binding ActiveItem}" DisplayMemberPath="DisplayName">
   <TabControl.ContentTemplate>
      <DataTemplate>
         <ContentControl s:View.Model="{Binding}" VerticalContentAlignment="Stretch" HorizontalContentAlignment="Stretch" IsTabStop="False"/>
      </DataTemplate>
   </TabControl.ContentTemplate>
</TabControl>
```

However, that's a bit of a mouthful, so Stylet provides a style for you which does the same thing. This means you can instead do:

---
><font color="#63aebb" face="微软雅黑">但是，这有点让人感到麻烦，所以 Style 为你提供了另一种实现方式。你可以这样做：</font>

```xml
<TabControl Style="{StaticResource StyletConductorTabControl}"/>
```


### `Conductor<T>.Collection.AllActive`

This conductor is very similar to `Conductor<T>.Collection.OneActive`, except that it doesn't have a single `ActiveItem`. Instead, it has only a collection of `Item`. When an item is activated (using `ActivateItem`), it's added to this collection, and when it's closed it's removed from this collection.

Calling `DeactivateItem` will deactivate the item in-place, without removing it from the `Items` collection.

The `Items` collection can also be manipulated directly. Any items added will be activated, and any removed will be closed.

A typical use-case might be using an ItemsControl, where all of the items are visible at once. A ViewModel which uses an ItemsControl in this way might look like this (again, see below for the short version):

---
><font color="#63aebb" face="微软雅黑">该 conductor 与 `Conductor<T>.Collection.OneActive` 非常相似，只是它没有 `ActiveItem`。它只有 Item 集合。当一个项目被激活（使用 `ActivateItem`）时，它被添加到这个集合中，当它被关闭时，它将被从这个集合中删除。

>调用 `DeactivateItem` 将直接停用该 item，而不将其从 `Items` 集合中删除。

>该 `Items` 集合也可以直接操作。添加的任何 item 都将被激活，任何已删除的 item 都将被关闭。

>典型的用例是使用 ItemsControl，其中所有项目一次可见。以这种方式使用 ItemsControl 的 ViewModel 如下所示（请参阅下面的简短版本）：</font>

```xml
<ItemsControl ItemsSource="{Binding Items}">
   <ItemsControl.ItemTemplate>
      <DataTemplate>
         <ContentControl s:View.Model="{Binding}" VerticalContentAlignment="Stretch" HorizontalContentAlignment="Stretch" IsTabStop="False"/>
      </DataTemplate>
   </ItemsControl.ItemTemplate>
</ItemsControl>
```

As this is quite verbose, Stylet provides a style which sets these properties for you:

---
><font color="#63aebb" face="微软雅黑">由于这非常冗长，Stylet 为你提供了一种更方便的方式：</font>

```xml
<ItemsControl Style="{StaticResource StyletConductorItemsControl}"/>
```

### `Conductor<T>.StackNavigation`

This conductor is something of a hybrid between `Conductor<T>` and `Conductor<T>.Collection.OneActive`, and it provides something extra: stack-based navigation.

It has a single `ActiveItem`, but also keeps a (private) history of the past items which were active. When you activate a new item, the previous `ActiveItem` is deactivated, and pushed onto the history stack. Calling `GoBack()` will close the current `ActiveItem`, and re-activate the top item from this history stack, and set it as the new `ActiveItem`.

If you call `CloseItem` on the current `ActiveItem`, this has the same effect. If you call `CloseItem` on any item which exists in the history stack, that item will be closed and removed from the history stack. Calling `Clear()` will close and remove all items from the history stack.

---
><font color="#63aebb" face="微软雅黑">该 conductor 是 `Conductor<T>` 和 `Conductor<T>.Collection.OneActive` 的混合体，它提供了额外的内容:基于堆栈方式的导航。

>它有一个 `ActiveItem` ，但也保存了过去活动的 item 的(私有)历史记录。当你激活一个新 item 时，先前的 `ActiveItem` 被停用，并被放入历史堆栈。调用 `GoBack()` 将关闭当前的 `ActiveItem`，从历史堆栈中重新激活最上面的 item，并将其设置为新的 `ActiveItem`。 

>如果在当前的 `ActiveItem` 上调用 `CloseItem`，效果是相同的。如果对历史堆栈中存在的任何项调用 `CloseItem`，该项 item 被关闭并从历史堆栈中删除。调用 `Clear()` 将关闭并从历史堆栈中删除所有 item。</font>

### `WindowConductor`

This one's a bit of an oddball, as it's internal and you're not expected to ever interact with it directly, but I've included it here for interest. Whenever you display a dialog or window using the `WindowManager` (this includes the window which Stylet shows when you first start your application), a new `WindowConductor` manages its lifecycle. Whenever your window or dialog is minimised, it's deactivated. Whenever it's maximized, it's activated. If your ViewModel requests that it be closed (see `RequestClose` above), the `WindowConductor` handles this. Similarly, if the user closes your window themselves, the `WindowConductor` will ask your ViewModel whether it's ready to be closed.

---
><font color="#63aebb" face="微软雅黑">当你使用 `WindowManager`（包括 Stylet 首次启动应用程序时显示的窗口）显示对话框或窗口时，新的 `WindowConductor` 管理其生命周期。当窗口或对话框最小化时，会被停用。最大化时，会被激活。如果的 ViewModel 请求关闭（见 `RequestClose` 上文），则 `WindowConductor` 会进行处理。同样，如果用户关闭窗口，`WindowConductor` 则会询问 ViewModel 是否关闭。</font>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[BindableCollection - 绑定集合](./BindableCollection.md)