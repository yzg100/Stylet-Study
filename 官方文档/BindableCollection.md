Overview - 概述
--------

`BindableCollection<T>` is a subclass of [`ObservableCollection<T>`](http://msdn.microsoft.com/en-us/library/ms668604%28v=vs.110%29.aspx). It's the class to use if you have a collection of something in your ViewModel, and want to use it as the `ItemsSource` / etc for something in your View (and have the View be notified whenever an item is added to / removed from that collection).

However, it adds a couple of useful extra features:

 - New `AddRange`, `RemoveRange`, and `Refresh` methods
 - Is thread-safe

---
><font color="#63aebb" face="微软雅黑">`BindableCollection<T>` 是 [`ObservableCollection<T>`](http://msdn.microsoft.com/en-us/library/ms668604%28v=vs.110%29.aspx) 的子类。如果 ViewModel 中的内容，希望将它用作 `ItemsSource` 的数据源，并且在该集合中添加/删除 item 时都会通知 View 就可以使用此类。

>它增加了一些有用的功能：
>- `AddRange`, `RemoveRange` 和 `Refresh` 方法
>- 它是线程安全的</font>

New Methods - 新方法
-----------

`ObservableCollection<T>` is missing a couple of very useful methods: `AddRange` and `RemoveRange`.
These do pretty much what you'd expect, allowing you to add a remove a range of elements at once, without
having to manually iterate over each element and calling `collection.Add(element)` on each (while raising lots
of events for each element added). `AddRange` and `RemoveRange` will raise one set of events per range added/removed only.

`Refresh` is a convenience.
It does not modify the collection in any way, but does cause the `PropertyChanged` and `CollectionChanged` events to be fired, indicating to any UI elements that the collection has been modified and that they should reload their data.
It's not often needed, but when it is it's *really* needed.

---
><font color="#63aebb" face="微软雅黑">`ObservableCollection<T>` 缺少一些非常有用的方法：`AddRange` 和 `RemoveRange`。`BindableCollection<T>` 可以满足你，允许一次添加删除一系列元素，而无需手动迭代每个元素并调用每个元素 `collection.Add(element)`（同时为每个添加的元素引发事件）。`AddRange` 和 `RemoveRange` 只在添加/删除的范围内引发事件。

>`Refresh` 很方便。它不会以任何方式修改集合，但会导致触发 `PropertyChanged` 和 `CollectionChanged` 事件，向任何UI元素指示集合已被修改应该重新加载数据。它并不经常需要，但它真的需要它。</font>

Thread Safety - 线程安全
-------------

Thread safety is achieved by dispatching all actions (adds, removes, clear, reset, etc) to the UI thread. The dispatch uses `Execute.OnUIThreadSync`, which means that:

 - These operations are synchronous: the method being called won't return until the action has been completed.
 - They're free if you're already on the UI Thread - the operation will be carried out synchronously in this case.
 - All `PropertyChanged` and `CollectionChanged` events are always raised on the UI thread.

That last point means that there is no `PropertyChangedDispatcher` property on `BindableCollection<T>`, as there is with `PropertyChangedBase` - the event is always raised on the UI thread, since the operation the property relates to is always performed on the UI thread. Similarly, there's no `CollectionChangedDispatcher` concept.

---
><font color="#63aebb" face="微软雅黑">线程安全是通过向UI线程调度所有操作(添加、删除、清除、重置等)来实现的。调度使用`Execute.OnUIThreadSync`意味着：
操作是同步的：被调用的方法在操作完成之前不会返回。

>- 如果在空闲 UI 线程中，操作将同步执行。
>- 所有 `PropertyChanged` 和 `CollectionChanged` 事件总是在 UI 线程上引发。

>在 `BindableCollection<T>` 上没有 `PropertyChangedDispatcher` 属性，就像 `PropertyChangedBase` 一样 - 事件总是在 UI 线程上引发，因为属性关联的操作总是在 UI 线程上执行。同样，这也没有 `CollectionChangedDispatcher` 概念。</font>


[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Validation using ValidatingModelBase - 使用 ValidatingModelBase 验证](./ValidatingModelBase.md)