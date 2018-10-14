Overview
--------

`BindableCollection<T>` is a subclass of [`ObservableCollection<T>`](http://msdn.microsoft.com/en-us/library/ms668604%28v=vs.110%29.aspx). It's the class to use if you have a collection of something in your ViewModel, and want to use it as the `ItemsSource` / etc for something in your View (and have the View be notified whenever an item is added to / removed from that collection).

However, it adds a couple of useful extra features:

 - New `AddRange`, `RemoveRange`, and `Refresh` methods
 - Is thread-safe


New Methods
-----------

`ObservableCollection<T>` is missing a couple of very useful methods: `AddRange` and `RemoveRange`.
These do pretty much what you'd expect, allowing you to add a remove a range of elements at once, without
having to manually iterate over each element and calling `collection.Add(element)` on each (while raising lots
of events for each element added). `AddRange` and `RemoveRange` will raise one set of events per range added/removed only.

`Refresh` is a convenience.
It does not modify the collection in any way, but does cause the `PropertyChanged` and `CollectionChanged` events to be fired, indicating to any UI elements that the collection has been modified and that they should reload their data.
It's not often needed, but when it is it's *really* needed.


Thread Safety
-------------

Thread safety is achieved by dispatching all actions (adds, removes, clear, reset, etc) to the UI thread. The dispatch uses `Execute.OnUIThreadSync`, which means that:

 - These operations are synchronous: the method being called won't return until the action has been completed.
 - They're free if you're already on the UI Thread - the operation will be carried out synchronously in this case.
 - All `PropertyChanged` and `CollectionChanged` events are always raised on the UI thread.

That last point means that there is no `PropertyChangedDispatcher` property on `BindableCollection<T>`, as there is with `PropertyChangedBase` - the event is always raised on the UI thread, since the operation the property relates to is always performed on the UI thread. Similarly, there's no `CollectionChangedDispatcher` concept.
