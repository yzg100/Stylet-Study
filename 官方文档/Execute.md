Summary - 摘要
-------

`Execute` is a little static helper, which makes it easier to dispatch delegates to be run on the UI thread. It wraps `Application.Current.Dispatcher`, and provides methods to make it easier and terser to use.

It also provides a helper property, `Execute.InDesignMode`. This is true if and only if the Visual Studio or Expression Blend designer is active, and code is being executed to supply dummy data for design-time display.

A high-level summary of the methods it offers is given in the table below, with more in-depth explanations given after.

---
><font color="#63aebb" face="微软雅黑">`Execute` 是一个小的静态助手，这使得在 UI 线程上执行委托更加容易。它包装Application.Current.Dispatcher，并提供方法使其更容易使用。

>它还提供了一个辅助属性 Execute.InDesignMode。当且仅当 Visual Studio 或 Expression Blend 设计器处于活动状态并且正在执行代码以提供用于设计时显示的虚拟数据。

>下表给出了它提供的方法的摘要，之后给出了更深入的解释。</font>

Method - 方法                | Inline if possible - 可以内联 | Waits for Completion - 等待完成
----------------------------|:-------------------:|:--------------------:
Execute.OnUIThread          | ✔                   | ✘
Execute.OnUIThreadSync      | ✔                   | ✔ (Blocks)
Execute.OnUIThreadAsync     | ✔                   | ✔ (Task)
Execute.PostToUIThread      | ✘                   | ✘
Execute.PostToUIThreadAsync | ✘                   | ✔ (Task)

**Inline if possible:** The method will check if the current thread is the UI thread. If it is, the delegate will be run synchronously. If it isn't, then it will be dispatched to the UI thread in some form.

---
><font color="#63aebb" face="微软雅黑">`可以内联`：该方法将检查当前线程是否为 UI 线程。如果是，委托将同步执行。如果不是，那么它将以某种形式被分派到 UI 线程执行。</font>

**Waits for completion:** Either blocks until the delegate has finished executing, or returns a Task which completes when the delegate has finished executing.

---
><font color="#63aebb" face="微软雅黑">`等待完成`：在委托完成执行之前阻塞，或者返回在委托完成执行时完成的任务。</font>

Details - 细节
-------

#### Execute.OnUIThread
Checks to see whether the current thread is the UI thead. If it is, the delegate will be run synchronously. If it is not, the delegate will be dispatched to the UI thread, to be run at some point in the future. In this case, `Execute.OnUIThread` will not wait for the delegate to complete.

This mirrors the traditional pattern of:

---
><font color="#63aebb" face="微软雅黑">检查当前线程是否是 UI 线程。如果是，则委托将同步执行。如果不是，则委托将被分派到 UI 线程，以便未来某个时候执行。在这种情况下，Execute.OnUIThread 不会等待委托完成。

>传统模式： </font>

```csharp
public static void InvokeIfRequired(Action action)
{
    if (Application.Current.Dispatcher.CheckAccess())
        action();
    else
        Application.Current.Dispatcher.BeginInvoke(action);
}
```

#### Execute.OnUIThreadSync
Checks to see whether the current thread is the UI thread. If it is, then it will run the delegate synchronously. If it is not, then it will dispatch the delegate to be run on the UI thread, and block until it has finished executing.

It is therefore very similar to `Execute.OnUIThread`, except that it will only return after the delegate has finished executing.

---
><font color="#63aebb" face="微软雅黑">检查当前线程是否是 UI 线程。如果是，则它将同步执行委托。如果不是，那么它将调度委托到 UI 线程上执行，并阻塞直到它完成执行。

>因此它与 Execute.OnUIThread 非常相似，只是在委托完成执行后它才会返回。</font>

#### Execute.OnUIThreadAsync
Checks to see whether the current thread is the UI thread. If it is, then it will run the delegate synchronously, and return a completed Task. If it is not, then it will dispatch the delegate to be run on the UI thread at some point in the future, and return a Task which completes when the delegate has finished executing.

It is therefore effectively the async version of `Execute.OnUIThreadSync`.

---
><font color="#63aebb" face="微软雅黑">检查当前线程是否是 UI 线程。如果是，则它将同步执行委托，并返回已完成的任务。如果不是，那么它将调度委托在未来的某个时候在 UI 线程上执行，并返回一个在委托完成执行时完成的任务。

>因此它实际上是异步版本 `Execute.OnUIThreadSync`。</font>

#### Execute.PostToUIThread
Regardless of whether the current thread is the UI thread, will post the delegate to be run on the UI thread at some point in the future. 

---
><font color="#63aebb" face="微软雅黑">无论当前线程是否是 UI 线程，都会在未来的某个时刻发布在 UI 线程上执行委托。</font>

#### Execute.PostToUIThreadAsync
Regardless of whether the current thread is the UI thread, will post the delegate to be run on the UI thread at some point in the future, and returns a Task which completes when the delegate has finished executing.

**BEWARE** you must never do something like `Execute.PostToUIThreadAsync(() => something(foo)).Wait()`. If you do that from the UI thread, you will cause a deadlock. This approach does not make sense with the `Execute.PostXXX` methods - use `Execute.OnUIThreadSync` or `Execute.OnUIThreadAsync` instead.

---
><font color="#63aebb" face="微软雅黑">无论当前线程是否为 UI 线程，都将在未来的某个时候在 UI 线程上发布将要运行的委托，并返回在委托完成执行时完成的 Task。

>`注意`：你绝不能这样做 `Execute.PostToUIThreadAsync(() => something(foo)).Wait()`。如果从 UI 线程执行此操作，则会导致死锁。这种方法对于 `Execute.PostXXX` 没有意义 - 使用 `Execute.OnUIThreadSync` 或 `Execute.OnUIThreadAsync` 代替。</font>

Advanced: Unit Testing - 高级：单元测试
----------------------

### The Dispatcher

`Execute` actually has a level of abstraction from `Application.Current.Dispatcher`.
`Execute.Dispatcher` is a static property of type `IDispatcher`, and is used by `Execute` to dispatch delegates.
This property can never be null, and defaults to an `IDispatcher` implementation which executes everything synchronously. It is then overridden in `BootstrapperBase` to be a wrapper around `Application.Current.Dispatcher`.

This behaviour means that methods using one of the `Execute` methods can be unit-tested, or used at design-time.
In unit testing, all `Execute` methods will run their delegate synchronously (since a Dispatcher isn't available).

If you need to, you can also set `Execute.Dispatcher` to a custom `IDispatcher` implementation for your unit tests. 

---
><font color="#63aebb" face="微软雅黑">`Execute` 具有从 `Application.Current.Dispatcher` 中抽象的级别。`Execute.Dispatcher` 是类型为 `IDispatcher` 的静态属性，由 `Execute` 分派委托。该属性永远不能为 Null，默认为 `IDispatcher` 实现，它同步执行所有操作。然后，在 `BootstrapperBase` 中重写它，使其成为 `Application.Current.Dispatcher` 的封装。 

>此行为意味着使用 `Execute` 的方法可以进行单元测试，也可以在设计时使用。在单元测试中，所有 `Execute` 方法都将同步运行其委托（因为 Dispatcher 不可用）。

>如果需要，你还可以设置单元测试 `Execute.Dispatcher` 的自定义 `IDispatcher` 实现。</font>

### Design Mode - 设计模式

`Execute.InDesignMode` is also settable, and this will override the *"actual"* value.
It is anticipated that you will almost never need to actually do this, but sometimes it's unavailable in order to unit test queer little edge cases (there are a couple of such cases in Stylet).

---
><font color="#63aebb" face="微软雅黑">`Execute.InDesignMode` 也是可设置的，这将覆盖“实际”值。预计你永远都不需要这样做，但有时它不可用于单元测试奇怪的案例（在 Stylet 中有几个这样的案例）。</font>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Screens and Conductors - 屏幕和指挥](./Screens-and-Conductors.md)