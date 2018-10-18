Stylet is a minimal, but powerful, MVVM framework inspired by [Caliburn.Micro](http://caliburnmicro.com/). Its intention is to cut down on the complexity and magic even further, to allow people (coworkers) unfamiliar with any MVVM framework to get up to speed more quickly.

It also provides features not available in Caliburn.Micro, including its own IoC container, easy ViewModel validation, and even an MVVM-compatible MessageBox.

The low LOC count and very comprehensive test suite makes it an attractive option for projects where using and validating/verifying SOUP comes with a high overhead, which its modular toolkit-inspired architecture means it's easy to use just the bits you like, or replace the bits you don't.

A brief feature list is shown below. Follow the links on the right to learn more.

---
><font color="#63aebb" face="微软雅黑">Stylet 是一个受 [Caliburn.Micro](http://caliburnmicro.com/) 启发小巧但功能强大的 MVVM 框架。它的目的是进一步降低复杂度，让任何不熟悉 MVVM 框架的人快速的上手。

>它还提供了 Caliburn.Micro 中没有的功能，包括它自己的IoC容器，简单的 ViewModel 验证，甚至是与 MVVM 兼容的 MessageBox。很少的代码和全面的测试使其成为使用和验证/验证 SOUP 带来高开销的项目的一个不错的选择。模块化工具包，灵巧的架构意味着它很容易让你使用喜欢的部分，或者替换你不喜欢的部分。

>下面列出了一个简短的功能列表。请按照目录中的链接学习更多内容。

</font>

#### [ViewModel First - 视图模型优先](./ViewModel-First.md)

The classic MVVM structure, where a view knows how to instantiate its ViewModel, and ViewModels typically don't communicate directly, is known as View-first. However, reversing this pattern - instantiating the ViewModels yourself and having the Views automatically attached - provides many advantages, allowing you to compose your ViewModels in a way which should feel very familiar. This ViewModel-first approach is the only one supported by Stylet.

---
><font color="#63aebb" face="微软雅黑">典型的 MVVM 结构(View 知道如何实例化它的 ViewModel，而且 ViewModel 通常不直接通信)称为 “View-first”。然而，反转这种模式-自己实例化 ViewModel 并自动附加 View - 这提供了许多优点，允许你以一种非常熟悉的方式组合 ViewModels。这种 ViewModel-First 方法是 Stylet 支持的方法。</font>

#### [Actions - 活动](./Actions.md)

The ICommand interface used by WPF is powerful, but clunky when used in an MVVM architecture. It doesn't seem right that actions to be taken by your ViewModel in response to things like button clicks should be represented as properties, rather than methods. A simple `<Button Command="{s:Action DoSomething}"/>` will cause `DoSomething()` on your ViewModel to be called every time the button is clicked. Additionally, if you have a bool property called `CanDoSomething`, that will be observed and used to tell whether the button should be enabled or disabled.

Actions also work with events, allowing you to do things like `<Button MouseEnter="{s:Action DoSomethingElse}"/>`.

---
><font color="#63aebb" face="微软雅黑">WPF 的 ICommand 接口非常强大，但在 MVVM 体系结构中使用时却很笨拙。ViewModel 在响应按钮单击时所采取的操作应该表示为属性，而不是方法。一个简单的 `<Button Command="{s:Action DoSomething}"/>` 会在每次单击按钮时调用 ViewModel 上的 `DoSomething()`。另外，如果你有一个名为 `CanDoSomething` 的 bool 属性，它将被观察并用于判断按钮是否应该启用或禁用。

>Action 也适用于事件，允许你做类似的事情 `<Button MouseEnter="{s:Action DoSomethingElse}"/>`。
</font>

#### [Screens and Conductors - 屏幕和指挥](./Screens-and-Conductors.md)

The Screen class provides many things which make it an attractive base class for your ViewModels: PropertyChanged notifications, validation, and the ability to be notified when it's shown/hidden/closed, and the ability to control if and when it can be closed.

---
><font color="#63aebb" face="微软雅黑">Screen 类提供了许多使其成为 ViewModels 有吸引力的基类的功能：属性更改通知、验证、在显示 / 隐藏 / 关闭时得到通知，以及控制是否关闭和何时关闭的能力。 </font>

#### [The EventAggregator - 事件聚合器](./The-EventAggregator.md)

Stylet's Event Aggregator is very similar to Caliburn.Micro's, and allows subscribers to receive messages from publishes without either having knowledge of, or retaining, the other. This is particularly useful for messaging between ViewModels, although it has plenty of other uses.

---
><font color="#63aebb" face="微软雅黑">Stylet 的 Event Aggregator 与 Caliburn.Micro 非常相似，它允许订阅者在不知道或不保留发布信息的情况下接收发布的消息。这对于 ViewModels 之间的消息传递特别有用，当然它还有很多其他用途。</font>

#### [The WindowManager - 窗口管理器](./The-WindowManager.md)

With a ViewModel-first approach, you display windows and dialogs by referencing the ViewModel to display, and the View gets attached automatically. The WindowManager allows this to be done with ease.

An MVVM-compatible MessageBox implementation is also provided, so you don't have to roll your own.

---
><font color="#63aebb" face="微软雅黑">使用 ViewModel-first 方法，通过引用要显示的 ViewModel 来显示窗口和对话框，并自动附加 View。WindowManager 可以轻松完成此操作。

>还提供与 MVVM 兼容的 MessageBox 实现，你不必自己实现消息框。</font>
#### [Validation - 验证](./ValidatingModelBase.md)

Validation in MVVM is traditionally a bit of a pain: it requires a fair amount of boilerplate in each ViewModel that requires validation, and resources on how to do this well are sparse.

Stylet comes with a framework for taking your favourite validation library (e.g. [FluentValidation](https://fluentvalidation.codeplex.com/)) and handles running validations and reporting the results to the View.

---
><font color="#63aebb" face="微软雅黑">在传统的 MVVM 中验证比较麻烦:每个需要验证的 ViewModel 中都需要大量的引用，而如何做好这一点的资料却很少。

>Stylet 附带了一个框架，用于获取你最喜欢的验证库（例如: [FluentValidation](https://fluentvalidation.codeplex.com/)）并处理运行验证并将结果报告给 View。</font>

#### [StyletIoC-Introduction - StyletIoC 简介](./Ioc/StyletIoC-Introduction.md)

Stylet comes with its own lightweight and extremely fast (but still powerful) IoC container, although it's easy to use another one if you'd prefer.

---
><font color="#63aebb" face="微软雅黑">Stylet 拥有自己的轻量级的、非常快(但仍然很强大)的 IoC 容器，如果你愿意，你也可以很容易地使用另一个容器。</font>

#### MIT license

Stylet is distributed under the MIT license, which allows you to modify Stylet, and include it in commercial projects, without attributation (the only restriction being that you must include a copy of the license). I'm open to re-licensing it on a case-by-case basis if you require this.

---
><font color="#63aebb" face="微软雅黑">Stylet 根据 MIT许可证分发，允许你修改 Stylet，并将其包含在商业项目中，无需归属（唯一的限制是你必须包含许可证副本）。如果你需要，我可以根据具体情况重新授权。</font>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Quick Start - 快速入门](./Quick-Start.md)