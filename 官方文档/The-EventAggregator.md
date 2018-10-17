
The EventAggregator is a decentralised, weakly-binding, publish/subscribe-based event manager.

---
><font color="#63aebb" face="微软雅黑">EventAggregator 是一个分散化、弱绑定、基于发布/订阅的事件管理器。</font>

Publishers and Subscribers - 发布和订阅
--------------------------
<H3>Subscribers - 订阅</H3>

Subscribers interesting in a particular event can tell the IEventAggregator of their interest, and will be notified whenever a publisher publishes that particular event to the IEventAggregator.

Events are classes - do whatever you want with them. For example:

---
><font color="#63aebb" face="微软雅黑">在特定事件中感兴趣的订阅者可以告诉 IEventAggregator 他们感兴趣的事情，并且每当发布者将该特定事件发布到 IEventAggregator时，就会得到通知。

>事件是类 - 用它们做任何你想做的事情。例如：</font>

```csharp
class MyEvent { 
  // Do something 
}
```

Subscribers must implement `IHandle<T>`, where `T` is the event type they are interested in receiving (they can of course implement multiple `IHandle<T>`'s for multiple `T`'s). They must then get hold of an instance of the IEventAggregator, and subscribe themselves, for example:

---
><font color="#63aebb" face="微软雅黑">订阅者必须实现 `IHandle<T>`，其中 `T` 是他们感兴趣的接收事件类型(他们当然可以为多个 `T` 实现多个 `IHandle<T>`)。然后，他们必须获得IEventAggregator 的一个实例，并自己订阅，例如:</font>

```csharp
class Subscriber : IHandle<MyEvent>, IHandle<MyOtherEvent>
{
   public Subscriber(IEventAggregator eventAggregator)
   {
      eventAggregator.Subscribe(this);
   }

   public void Handle(MyEvent message)
   {
      // ...
   }

   public void Handle(MyOtherEvent message)
   {
      // ...
   }
}
```

For VB.NET users, the `Sub New()` passing the eventAggregator by reference will probably fail across namespaces, and can be irritating to have to define with each new subscriber. Thus, it may be easier to define your eventAggregator in a global module, then subscribe directly to it instead of passing its reference along to each new ViewModel you call. 

---
><font color="#63aebb" face="微软雅黑">VB.NET 用户通过引用传递 eventAggregator 的 `Sub New()` 可能会在名称空间之间失败，并且不得不对每个新订阅服务器进行定义，这可能令人恼火。因此，在全局模块中定义 eventAggregator 更容易，然后直接订阅它，而不是将其引用传递给你调用的每个新视图模型。</font>

```vb.net
Module Global
  Public eventAggregator as IEventAggregator
End Module

Class Subscriber : Implements IHandle(Of MyEvent)

  Public Sub New()
  Global.eventAggregator.Subscribe(Me)
  End Sub
  
  'Public Sub Handle...

End Class
```
Make sure to keep the namespace for the *module* blank, so that it can be used throughout the program. 

---
><font color="#63aebb" face="微软雅黑">确保将模块的命名空间保留为空，以便可以在整个程序中使用它。</font>

<H3>Publishers - 发布</H3>

Publishers must also get an instance of the IEventAggregator, but they don't need to subscribe themselves - they only need to call IEventAggregator.Publish every time they want to publish an event, for example:

---
><font color="#63aebb" face="微软雅黑">发布者还必须获得 IEventAggregator 的实例，但他们不需要自己订阅 - 每当他们想要发布一个事件时他们只需要调用IEventAggregator.Publish。例如:</font>

```csharp
class Publisher
{
   private IEventAggregator eventAggregator;
   public Publisher(IEventAggregator eventAggregator)
   {
      this.eventAggregator = eventAggregator;
   }

   public void PublishEvent()
   {
      this.eventAggregator.Publish(new MyEvent());
   }
}
```

Again, for VB.NET users, if you've set up the global module then you don't need to pass the eventAggregator to the Publisher. You can just publish directly to the global eventAggregator;

---
><font color="#63aebb" face="微软雅黑">同样，对于 VB.NET 用户，如果你已设置全局模块，则无需将 eventAggregator 传递给 Publisher。你可以直接发布到全局 eventAggregator;</font>

```vb.net
Class Publisher

  Public Sub PublishEvent()
  Global.eventAggregator.Publish(New MyEvent())
  End Sub
  
End Class
```
Unsubscribing and weak binding - 取消订阅和弱绑定
------------------------------

Because the IEventAggregator is weakly binding, subscribers don't need to unsubscribe themselves - the IEventAggregator won't retain them. It is however possible for a subscriber to unsubscribe itself if it wants - call &nbsp;

---
><font color="#63aebb" face="微软雅黑">因为 IEventAggregator 的绑定能力很弱，所以订阅者不需要自己取消订阅 — IEventAggregator 不会保留他们。然而，如果订阅方想要取消订阅，则可以调用。</font>

```csharp
IEventAggregator.Unsubscribe(this);
```


Publishing synchronously and asynchronously - 同步和异步发布
-------------------------------------------

The default `IEventAggregator.Publish` method publishes the event synchronously. You can also call `PublishOnUIThread` to dispatch asynchronously to the UI thread, or `PublishWithDispatcher` and pass any action you want to act as the dispatcher (this can be useful if writing your own methods on IEventAggregator).

---
><font color="#63aebb" face="微软雅黑">默认 `IEventAggregator.Publish` 方法同步发布事件。你还可以调用 `PublishOnUIThread` 异步调度到 UI 线程，或者 `PublishWithDispatcher` 传递你想要充当调度程序的任何操作（在  IEventAggregator 上编写自己的方法，这很有用）。</font>

Channels - 通道
--------

Subscribers can listen to particular channels, and publishers can publish events to particular channels. If an event is published to a particular channel, only subscribers who have subscribed to that channel will receive that event. This can be useful if the same message type is used in several different contexts.

Channels are strings, and so allow loose coupling between subscribers to a channel, and publishers to that channel.

By default, `Subscribe()` will subscribe the subscriber to a single channel, `EventAggregator.DefaultChannel`. Similarly, `Publish()` (and all variants thereof) will publish events to that same default channel. However, you can specify your own channel(s) if you want....

---
><font color="#63aebb" face="微软雅黑">订阅者可以收听特定频道，发布者可以将事件发布到特定频道。如果事件发布到特定频道，则只有订阅该频道的订阅者才会收到该事件。如果在多个不同的上下文中使用相同的消息类型，这很有用。

>频道是字符串，因此允许频道的订阅者和该频道的发布者之间的松散耦合。

>默认情况下，`Subscribe()` 将订阅者订阅到单个频道，`EventAggregator.DefaultChannel`。同样，`Publish()`（及其所有变体）将事件发布到相同的默认通道。但是，如果需要，你可以指定自己的频道....</font>

### Subscribing to channels - 频道订阅

To subscribe to a particular channel, pass it as a parameter to `Subscribe`: `eventAggregator.Subscribe(this, "ChannelA")`. You can also subscribe to multiple channels: `eventAggregator.Subscribe(this, "ChannelA", "ChannelB")`.

In both of these cases, you will *not* be subscribed to `EventAggregator.DefaultChannel` - only to the channels listed. You will only receive events that were pushed to "ChannelA" or "ChannelB".

---
><font color="#63aebb" face="微软雅黑">要订阅特定频道，请将其作为参数传递给 `Subscribe`：`eventAggregator.Subscribe(this, "ChannelA")`。你还可以订阅多个频道：`eventAggregator.Subscribe(this, "ChannelA", "ChannelB")`。

>在这两种情况下，你都不会订阅到 `EventAggregator.DefaultChannel`。你只会收到被推送到 "ChannelA" 或 "ChannelB" 的事件。</font>

### Publishing to channels - 发布到频道

To publish to a particular channel, pass it as a parameter to `Publish`: `eventAggregator.Publish(message, "ChannelA")`, or `eventAggregator.PublishOnUIThread(message, "ChannelA", "ChannelB")`, etc. As with subscribing above, the event will be published to all of the channels named, and not to the default channel.

---
><font color="#63aebb" face="微软雅黑">要发布到特定频道，请将其作为参数传递给 `Publish`：`eventAggregator.Publish(message, "ChannelA")`，或 `eventAggregator.PublishOnUIThread(message, "ChannelA", "ChannelB")`。与上面的订阅一样，事件将发布到所有已命名的频道，而不是默认频道。</font>

### Unsubscribing from channels 取消订阅频道

To unsubscribe from a channel, pass it to `Unsubscribe`: `eventAggregator.Unsubscribe(this, "ChannelA")`. You will remain subscribed to any other channels you were previously subscribed to, and have not unsubscribed from.

Calling `eventAggregator.Unsubscribe(this)` will unsubscribe you from *all* channels.

---
><font color="#63aebb" face="微软雅黑">要取消订阅频道，请将其传递给 `Unsubscribe`：`eventAggregator.Unsubscribe(this, "ChannelA")`。你之前订阅的其它频道，不会被取消订阅。</font>

Using your own IoC container - 使用你自己的IoC容器
----------------------------

If you're using StyletIoC with the default `Bootstrapper<TRootViewModel>`, you don't need to worry about this - the EventAggregator is set up correctly by default.

If you're using another IoC container, however, you need to make sure that the EventAggregator is registered as a singleton service to the interface `IEventAggregator` - there must be only one instance of the EventAggregator created, ever, and this once instance must be returned every time it is requested.

---
><font color="#63aebb" face="微软雅黑">如果你使用默认的 StyletIoC，则 `Bootstrapper<TRootViewModel>` 无需担心这一点 - 默认情况下是正确设置的。

>如果你使用其它 IoC 容器，则需要确保将 EventAggregator 注册为单例模式 `IEventAggregator` - 必须确保只创建一个 EventAggregator 实例，并且每次调用都返回的是该实例。</font>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[PropertyChangedBase - 属性通知基类](./PropertyChangedBase.md)