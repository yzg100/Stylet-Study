Screens and Conductors are a simple topic, but one which requires a few mental leaps, and requires you to cover all parts of them before they'll make sense. Trust me, it's well worth your time to read this article - they're hugely powerful, and well worth the time investment.

ViewModel Lifecycles
--------------------

A good place to start at is by looking at ViewModel lifecycles.

Imagine a tabbed interface - something like Visual Studio, which has (very simplistically) a shell (containing the menus, toolbars, etc) and a TabControl containing your editor tabs. In Stylet, each of those editor tabs would be backed by its own ViewModel.

Now, one of those ViewModels will start its life by being instantiated. Next up, it will be displayed. After that, it might be shown or hidden depending on which tab is currently active, before finally being closed. Just before it's closed it has the opportunity to prevent that closure to prompt you to save the file.

This, in a nutshell, is a ViewModel's lifecycle: it's created, then Activated (displayed to the user). After that, it can be Deactivated (still alive but not shown) and Activated again any number of times, before finally being Closed (after being asked whether it's ready to be closed).


IDisposable
-----------

It's worth noting that if a ViewModel implements `IDisposable`, then it will be disposed after it's closed by its parent (unless the parent's `DisposeChildren` property is false).


Introducing Conductors
----------------------

Now, a ViewModel doesn't magically know when it's been shown, hidden, or closed. It has to be told. This is the role of a Conductor.

A Conductor is, simply, a ViewModel which owns another ViewModel, and knows how to manage its lifecycle.

In our Visual Studio example, the Conductor is going to be the ViewModel which owns the TabControl inside which the editor ViewModels are displayed, so probably the Shell ViewModel. Whenever the user selects a  new editor tab, the Conductor is going to Deactivate the old tab, and Activate the new tab. When the user closes a tab, the Conductor is going to tell that tab that it's been Closed, then decide which is the next tab to display, and Activate that.

And that's it, really. ViewModels have a lifecycle, and that's implemented by the Conductor which owns the ViewModel.

This has been quite abstract so far - let's go into the details.


IScreen and Screen
------------------

As we saw above, a ViewModel's lifecycle is managed by a Conductor calling methods on that ViewModel. Those methods are defined in a set of independent interfaces - if you implement the interface, and that ViewModel's managed be a Conductor, that method will get called. You can pick and choose which interfaces you want, if you want.

There's an overarching interface called `IScreen` which composes them all, and a default implementation called `Screen`. This behaves really very well, and you'll probably never need to implement your own - but you can if you want.

 - `IScreenState`: Used to activate, deactivate, and close the ViewModel. Has `Activate`, `Deactivate`, and `Close` methods method, as well as events and properties to track changes in the screen's state.
 - `IGuardClose`: Used to ask the ViewModel whether it can close. Has a `CanCloseAsync` method.
 - `IViewAware`: Sometimes a ViewModel needs to know about its View (when it's attached, what it is, etc). This interface allows that through a `View` property and an `AttachView` method.
 - `IHaveDisplayName`: Has a `DisplayName` property. This name is used as the title for windows and dialogs displayed using [[The WindowManager]], and is also useful for things like TabControls.
 - `IChild`: It can be advantageous for a ViewModel to know what Conductor is managing it (to request that it be closed, for example). If the ViewModel implements `IChild`, it will be told this.

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


Conductors in Detail
-------------------

Conductors come in a variety of flavours, each of which has its own use-cases. A conductor can own a single ViewModel (think navigation with one page shown at a time), or multiple ViewModels. Those with multiple ViewModels can have have just one active at a time (think the TabControl from the Visual Studio example above), or all of them (think grid with lots of independent elements). Conductors can also add behaviour such as keeping a record of which ViewModels they've shown (useful for navigation). 

As with the `Screen` class, Stylet defines a number of interfaces which are of interest to conductors, and a number of implementations (depending on the sort of conductor behaviour you want), although you can of course implement your own.

The primary interface is `IConductor<T>`, which represents a conductor as you would interact with it. It has the following methods
 - `ActivateItem(T item)`: Take the given item, and activate it. Whether this deactivates a previous item is determined by the conductor.
 - `DeactivateItem(T item)`: Take the given item, and deactivate it. Whether this activates another item is determined by the conductor.
 - `CloseItem(T item)`: Take the given item, and close it. Whether this causes another item to be activated to take its place is conductor-specific.

The conductors which have a single active item (regardless of how many inactive items they may have) also implement `IHaveActiveItem<T>`, which has the single property `ActiveItem`.

All of the built-in conductors will set an item's `Parent` property to itself, if the item implements `IChild`. All of the built-in conductors additionally implement `IChildDelegate`, which allows the child to request that it be closed (by calling `CloseItem`). In the default `Screen` implementation, calling `Screen.RequestClose` will cause the screen to call `CloseItem` on its parent (provided its parent implement `IChildDelegate`), which in turn causes its parent (if it exists) to close it.


Built-In Conductors
-------------------

Stylet comes with some conductors built-in, which perform in a number of intuitive ways.

All of these conductors derive from `Screen`, allowing conductors to easily own other conductors. This means you can compose your conductors and screens in any way you want.

### `Conductor<T>`

This very simple conductor owns a single ViewModel (of type `T`), which is exposed as the `ActiveItem`. The `ActivateItem` method is used to replace the current `ActiveItem` with a new ViewModel instance, and will activate the new item and close the old one. Whenever the `Condcutor<T>` is activated, it activates its `ActiveItem`; likewise it deactivates and closes its `ActiveItem` when it is deactivated or closed, respectively.

When asked whether it can be closed (when `CanCloseAsync` is called), it returns whatever its `ActiveItem` returned, or true if there's no `ActiveItem`.

The `ActiveItem` can also be set directly, which has the same effect as calling `ActivateItem` on it.

A ViewModel for this Conductor will look something like this - a ContentControl bound to the conductor's ActiveItem:

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

```xml
<TabControl Style="{StaticResource StyletConductorTabControl}"/>
```


### `Conductor<T>.Collection.AllActive`

This conductor is very similar to `Conductor<T>.Collection.OneActive`, except that it doesn't have a single `ActiveItem`. Instead, it has only a collection of `Item`. When an item is activated (using `ActivateItem`), it's added to this collection, and when it's closed it's removed from this collection.

Calling `DeactivateItem` will deactivate the item in-place, without removing it from the `Items` collection.

The `Items` collection can also be manipulated directly. Any items added will be activated, and any removed will be closed.

A typical use-case might be using an ItemsControl, where all of the items are visible at once. A ViewModel which uses an ItemsControl in this way might look like this (again, see below for the short version):

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

```xml
<ItemsControl Style="{StaticResource StyletConductorItemsControl}"/>
```

### `Conductor<T>.StackNavigation`

This conductor is something of a hybrid between `Conductor<T>` and `Conductor<T>.Collection.OneActive`, and it provides something extra: stack-based navigation.

It has a single `ActiveItem`, but also keeps a (private) history of the past items which were active. When you activate a new item, the previous `ActiveItem` is deactivated, and pushed onto the history stack. Calling `GoBack()` will close the current `ActiveItem`, and re-activate the top item from this history stack, and set it as the new `ActiveItem`.

If you call `CloseItem` on the current `ActiveItem`, this has the same effect. If you call `CloseItem` on any item which exists in the history stack, that item will be closed and removed from the history stack. Calling `Clear()` will close and remove all items from the history stack.


### `WindowConductor`

This one's a bit of an oddball, as it's internal and you're not expected to ever interact with it directly, but I've included it here for interest. Whenever you display a dialog or window using the `WindowManager` (this includes the window which Stylet shows when you first start your application), a new `WindowConductor` manages its lifecycle. Whenever your window or dialog is minimised, it's deactivated. Whenever it's maximized, it's activated. If your ViewModel requests that it be closed (see `RequestClose` above), the `WindowConductor` handles this. Similarly, if the user closes your window themselves, the `WindowConductor` will ask your ViewModel whether it's ready to be closed.

