Introduction - 介绍
------------

Imagine the scene... The user is filling out a form you've painstakingly written, and they enter their name where they should have entered their email address. You need to detect this, and display the problem in a clear way.

Input validation is a big area, and there are many ways to go about it. The easiest and most appealing is to throw an exception in the setter for your property, like this:

---
><font color="#63aebb" face="微软雅黑">想象一下这个场景.用户正在填写一张表格，他们输入名字，输入电子邮件地址。你需要发现填写的问题，并以一种清晰的方式显示问题。 

>输入验证有很多方法可以解决它。最简单和最吸引人的是在你的属性的赋值时抛出异常，如下所示：</font>

```csharp
private string _name;
public string Name
{
   get { return this._name; }
   set
   {
      if (someConditionIsFalse)
         throw new ValidationException("Message");
      this._name = value;
   }

```

When the binding sets this property, it notices if an exception is thrown, and updates the validation state of the control appropriately.

This, however, ends up being a **thoroughly bad idea**. It means that your property can *only* be validated when it's set (you can't go through and validate the whole form when the user clicks 'Submit', for example), and it leads to big fat property setters with lots of duplicated logic. Horrible.

C# also defines two interfaces, both of which WPF knows about: [IDataErrorInfo](http://msdn.microsoft.com/en-gb/library/system.componentmodel.idataerrorinfo.aspx) and [INotifyDataErrorInfo](http://msdn.microsoft.com/en-us/library/vstudio/system.componentmodel.inotifydataerrorinfo). Both of these provide a means for the ViewModel to tell the View, through events and PropertyChanged notifications, that one or more properties have one or more validation errors. Of these, INotifyDataErrorInfo is newer, easier to use, and allows for asynchronous validation.

However, driving INotifyDataErrorInfo is still a bit unintuitive: it allows you to broadcast the fact that one or more properties have errors, but provides no easy means for you to run your validations, and requires that you keep a record of what errors are associated with which properties.

ValidatingModelBase aims to solve this, and provide an intuitive and easy way of running and reporting your validations.

---
><font color="#63aebb" face="微软雅黑">当绑定为属性赋值时，它会通知是否抛出异常，并相应地更新控件的验证状态

>然而，这不是好的方法。这意味着你的属性只能在赋值时进行验证（例如，当用户点击 "提交" 时，你无法通过并验证整个表单），并且会导致具有大量重复逻辑的属性赋值。

>C＃ WPF 中还定义了两个接口：[IDataErrorInfo](http://msdn.microsoft.com/en-gb/library/system.componentmodel.idataerrorinfo.aspx) 和 [INotifyDataErrorInfo](http://msdn.microsoft.com/en-us/library/vstudio/system.componentmodel.inotifydataerrorinfo)。这两者都为 ViewModel 提供了一种方法，通过事件和  PropertyChanged通知 View 一个或多个属性有验证错误。其中，INotifyDataErrorInfo 更新，更易于使用，并允许异步验证。

>但是，使用 INotifyDataErrorInfo 仍然不太直观：它允许你广播一个或多个属性有错误，但未提供运行验证的简单方法，且要求你记录与之关联的错误是哪些属性。

>ValidatingModelBase 旨在解决这个问题，并提供一种直观、简单的方式来运行和报告验证。</font>


ValidatingModelBase
-------------------

ValidatingModelBase derives from [PropertyChangedBase](./PropertyChangedBase.md), and is inherited by Screen. It builds on PropertyChangeBase's ability to notice when properties have changed to run and report your validations.

---
><font color="#63aebb" face="微软雅黑">ValidatingModelBase 派生自 [PropertyChangedBase](./PropertyChangedBase.md)，由 Screen 继承。它建立在 PropertyChangeBase 的基础上，能够注意属性何时更改，执行验证运行并报告验证结果。</font>

### IModelValidator

There are many ways to run validations, and many good libraries out there to help you. It isn't Stylet's intention to provide another validation library, so instead Stylet allows you to provide your own validation library to be used by ValidatingModelBase.

This manifests itself in ValidatingModelBase's `validator` property, which is an `IModelValidator`. The intention is that you write your own implementation of `IModelValidator`, which wraps your preferred validation library (I'll cover some examples of how to do this later), so that it can be used by ValidatingModelBase.

This interface has two important methods:

---
><font color="#63aebb" face="微软雅黑">有很多方法可以运行验证，还有很多好的库可以帮助你。Stylet 不打算提供另一个验证库，因此 Stylet 允许你提供自己的验证库以供 ValidatingModelBase 使用。

>这体现在 ValidatingModelBase 的 `validator` 属性中，它是 `IModelValidator`。自己编写 `IModelValidator` 实现，它封装了你喜欢的验证库(稍后我将介绍一些如何实现的示例)，以便验证模型库可以使用它。

>该接口有两个重要方法：</font>

```csharp
Task<IEnumerable<string>> ValidatePropertyAsync(string propertyName);
Task<Dictionary<string, IEnumerable<string>>> ValidateAllPropertiesAsync();
```

The first one is called by ValidatingModelBase when it wants to validate a single property by name, and returns an array of validation errors. The second one is called by ValidatingModelBase when you ask it to do a full validation, and returns a Dictionary of `property name => array of validation errors`. 

The fact that these methods are asynchronous allows you to take advantage of `INotifyDataErrorInfo`'s asynchronous validation capabilities, and run your validations on some external service if you wish. However, it's expected that most implementations of this interface will just return a completed Task.

There's also a third method:

---
><font color="#63aebb" face="微软雅黑">当 ValidatingModelBase 根据名称验证单个属性并返回验证错误数组时，调用第一个。在需要 ValidatingModelBase 进行完整验证时调用第二个方法，它返回的是属性名称对应的验证错误组字典。

>这些方法是异步的，这让你可以利用 `INotifyDataErrorInfo` 的异步验证功能，并在某些外部服务上运行你的验证。但是，这个接口的大多数实现都只返回一个已完成的任务。

>还有第三种方法：</font>

```csharp
void Initialize(object subject);
```

This is called by ValidatingModelBase when it is setting up its validation for the first time, and it passes in an instance of itself. This allows the implementation of `IModelValidator` to specialise itself for validating that particular instance of ValidatingModelBase. This has more relevance when we tie things into StyletIoC, see later.

There's also a generic version of this interface, `IModelValidator<T>`, which merely extends `IModelValidator`, and adds nothing extra. This is, again, useful when IoC contains come into the picture - more on that later.

---
><font color="#63aebb" face="微软雅黑">这是由 ValidatingModelBase 在第一次设置验证时调用的，并且它传入自身的实例。允许 `IModelValidator` 针对特定实例实现验证模型库。当我们将实际与 StyletIoC 联系起来时，这更具相关性，稍后会看到。

>还有这个接口的通用版本 `IModelValidator<T>` ，它只是扩展 `IModelValidator`，并没有额外增加。当 IoC 包含进来时是有用的 - 稍后会详细介绍。</font>

### Running Validations - 运行验证

First, you have to remember to pass your `IModelValidator` implementation to `ValidatingModelBase`. You can do this either by setting the `validator` property, or by calling an appropriate constructor:

---
><font color="#63aebb" face="微软雅黑">首先，你必须记住将 `IModelValidator` 实现传递给 `ValidatingModelBase`。你可以通过设置 `validator` 属性或者调用合适的构造函数来实现：</font>

```csharp
public class MyViewModel : ValidatingModelBase
{
   public MyViewModel(IModelValidator validator) : base(validator)
   {
   }
}
```

By default, ValidatingModelBase will run the validations for a property whenever that property changes (provided you call `SetAndNotify`, use `NotifyOfPropertyChange`, or use `PropertyChanged.Fody` to raise a PropertyChanged notification using the mechanisms defined in [PropertyChangedBase](./PropertyChangedBase.md)). It will then report any changes in the validation state of that property using the mechanisms defined in the `INotifyDataErrorInfo` interface. It will also change the value of the `HasErrors` property.

If you want to disable this auto-validation behaviour, set the `AutoValidate` property to `false`.

You can manually run validations for a single property by calling `ValidateProperty("PropertyName")`, or `ValidateProperty(() => this.PropertyName)`, if you want. There are also asynchronous versions of these if your validations are asynchronous - more on this later. If you want to validate a single property whenever it is set, you can do something like this:

---
><font color="#63aebb" face="微软雅黑">默认情况下，当属性发生更改时，ValidatingModelBase 将运行属性的验证(假设你调用 `SetAndNotify`，使用 `NotifyOfPropertyChange` 或使用 `PropertyChanged.Fody` 来使用 [PropertyChangedBase](./PropertyChangedBase.md)) 中定义的机制引发 PropertyChanged 通知。它将使用 `INotifyDataErrorInfo`  接口中定义的机制报告该属性验证状态的更改。还会改变 `HasErrors` 属性的值。

>如果要禁用自动验证， `AutoValidate` = `false`。

>你可以通过调用 `ValidateProperty("PropertyName")` 或 `ValidateProperty(() => this.PropertyName)` 手动执行单个属性的验证。如果你的验证是异步的，那么还会有异步版本——稍后会详细介绍。如果你想在赋值单个属性时验证它，你可以这样做:</font>

```csharp
private string _name
public string Name
{
   get { return this._name; }
   set
   {
      SetAndNotify(ref this._name, value);
      ValidateProperty();
   }
}
```

Additionally, you can run validations on all properties by calling `Validate()`. 

If you wish to run some custom code whenever the validation state changes (any property's validation errors change), override `OnValidationStateChanged()`.

---
><font color="#63aebb" face="微软雅黑">另外，你可以通过调用' Validate() '在所有属性上运行验证。

>如果你希望在验证状态发生变化时运行一些自定义代码(任何属性的验证错误发生变化)，请重写 `OnValidationStateChanged()`。</font>

Understanding and Using IModelValidator - 理解和使用 IModelValidator
---------------------------------------

In the next couple of sections, I'm going to take you through an example of implementing validation, using the very useful [FluentValidation](http://fluentvalidation.codeplex.com/) library.

FluentValidation works by you creating a new class, which implements `IValidator<T>` (you'll usually do this by extending `AbstractValidator<T>`, and which can validate a particular sort of model `T`). You then create a new instance of this, and use it to run your validations. So for example if you have a `UserViewModel`, you'll define a `UserViewModelValidator` which extends `AbstractValidator<UserViewModel`, and therefore implements `IValidator<UserViewModel>`, like this:

---
><font color="#63aebb" face="微软雅黑">在接下来的几节中，我将通过实现验证的例子，使用[FluentValidation](http://fluentvalid.codeplex.com/)库。

>FluentValidation 通过创建新类来实现 `IValidator<T>` (通过扩展 `AbstractValidator<T>` 来实现这一点，它可以验证一种特定的类型`T`)。然后，创建新实例，并使用它来运行验证。例如，如果你有 `UserViewModel`，你将定义 `UserViewModelValidator`，它扩展了`AbstractValidator<UserViewModel>`，因此实现了 `IValidator<UserViewModel>`，如下所示:</font>

```csharp
public class UserViewModel : Screen
{
   private string _name;
   public string Name
   {
      get { return this._name; }
      set { SetAndNotify(ref this._name, value); }
   }
}

public class UserViewModelValidator : AbstractValidator<UserViewModel>
{
   public UserViewModelValidator()
   {
      RuleFor(x => x.Name).NotEmpty();
   }
}
```

If we were using the `UserViewModelValidator` directly (without the help of ValidatingModelBase), we'd do something like:

---
><font color="#63aebb" face="微软雅黑">如果直接使用 `UserViewModelValidator` (不需要 ValidatingModelBase 的辅助)，我们会这样做：</font>

```csharp
public UserViewModel(UserViewModelValidator validator)
{
   this.Validator = validator;
}
// ...
this.Validator.Validate(this);
```

However, the point of using ValidatingModelBase is that it will automate running and reporting the validations. As discussed earlier, we'll need to wrap up our `UserViewModelValidator` in a way that ValidatingModelBase know how to interact with.

The easiest way to doing this is to write an adapter which can take any implementation of `IValidator<T>` (i.e. any custom validator which you've written), and expose it in a way which ValidatingModelBase understands. In case you've got lost, I'll run over the intended class hierarchies again:

- ValidatingModelBase.Validator is an IModelValidator
- UserViewModelValidator is an IValidator<UserViewModel>
- We will write an adapter, FluentValidationAdapter<T>, which is an IModelValidator
- FluentValidationAdapter<T> will accept an IValidator<T>, and wrap it up so that it can be accessed through IModelValidator
- Therefore, FluentValidationAdapter<UserViewModel> will take a UserViewModelValidator, and expose it as an IModelValidator;

Make sense so far? This may sound like a lot of work, but we can get our IoC container to do most of the heavy lifting, as we'll see soon.

Now, what will this look like in practice? First, remember when I said that `IModelValidator<T>` is defined as an interface which just implements `IModelValidator`? I'm not going to show you why just yet, but keep in mind that they're basically synonyms.

---
><font color="#63aebb" face="微软雅黑">然而，使用 ValidatingModelBase 的目的是使运行和报告验证过程自动化。正如前面所讨论的，我们需要用一种验证模型库知道如何交互的方式来包装 `UserViewModelValidator`。

>最简单的方法是编写一个适配器，它可以接受 `IValidator<T>` 的任何实现(即你编写的任何定制验证器)，并以验证模型库理解的方式公开它。如果你迷路了，我会再次遍历目标类层次结构：

>- ValidatingModelBase.Validator 是 IModelValidator
>- UserViewModelValidator 是 IValidator<UserViewModel>
>- 我们将编写适配器 FluentValidationAdapter<T>，它是 IModelValidator
>- FluentValidationAdapter 将接受 I​​Validator，并将其包装起来，以便可以通过 IModelValidator 访问它
>- 因此，FluentValidationAdapter 将采用 UserViewModelValidator，并将其公开为 IModelValidator;

>到目前为止有意义吗?这听起来似乎要做很多工作，但是我们可以让 IoC 容器来完成大部分繁重的工作，很快就会看到。

>那么，这在实践中会是什么样子？还记得我说过的 IModelValidator<T> 被定义为刚刚实现的接口 IModelValidator 吗？我不打算告诉你为什么，但请记住，它们基本上是相同的。</font>

```csharp
// Define the adapter - 定义适配器
public class FluentValidationAdapter<T> : IModelValidator<T>
{
   public FluentValidationAdapter(IValidator<T> validator)
   {
      // Store the validator - 存储验证
   }

   // Implement all IModelValidator methods, using the stored validator
   // 使用存储的验证实现所有 IModelValidator 方法
}

// This implements IValidator<UserViewModel>
// 实现 IValidator<UserViewModel>
public class UserViewModelValidator : AbtractValidator<UserViewModel>
{
   public UserViewModelValidator()
   {
      // Set up validation rules
      // 设置验证规则
   }
}

public class UserViewModel
{
   public UserViewModel(IModelValidator<UserViewModel> validator) : base(validator)
   {
      // ...
   }
}
```

There! If we were going to instantiate a new `UserViewModel` by hand, we'd do this:

---
><font color="#63aebb" face="微软雅黑">如果我们要手动实例化 `UserViewModel`，我们会这样做：</font>

```csharp
var validator = new UserViewModelValidator();
var validatorAdapter = new FluentValidationAdapter<UserViewModel>(validator);
var viewModel = new UserViewModel(validatorAdapter);
```

However, we can configure the IoC container to do this for us. This assumes you're using StyletIoC, although other containers can be configured similarly.

In your `ConfigureIoC` override in your bootstrapper, first tell StyletIoC to return a `FluentValidationAdapter<T>` whenever you ask for an `IModelValidator<T>`, by doing this:

---
><font color="#63aebb" face="微软雅黑">我们可以配置IoC容器来为我们执行此操作。假设你正在使用 StyletIoC，尽管其他容器也可以进行类似的配置。

>在 bootstrapper 的 `ConfigureIoC` 重载中，首先告诉 StyletIoC 在请求 `IModelValidator<T>` 时返回`FluentValidationAdapter<T>`，方法如下:</font>

```csharp
builder.Bind(typeof(IModelValidator<>)).To(typeof(FluentValidationAdapter<>));
```

So, whenever StyletIoC creates a new `UserViewModel`, it will realise that it needs an `IModelValidator<UserViewModel>`. It knows that it's been told how to create an `IModelValidator<T>` - by instantiating a new `FluentValidationAdapter<T>`. So it will try and create a new `FluentValidationAdapter<UserViewModel>`, see that *that* requires a new `IValidator<UserViewModel>`, and fall over because it can't find one.

Therefore, we need to tell StyletIoC how to create a new `IValidator<UserViewModel>`. We *could* do this the long way, by doing this:

---
><font color="#63aebb" face="微软雅黑">当 StyletIoC 创建新的 `UserViewModel` 时，它就会意识到需要 `IModelValidator<UserViewModel>`。它知道如何通过实例化新的 `FluentValidationAdapter<T>` 来创建 `ModelValidator<T>`。因此，它将尝试创建新的 ` FluentValidationAdapter<UserViewModel>`，需要新的 `IValidator<UserViewModel>`，因为找不到。

>因此，我们需要告诉 StyletIoC 如何创建新的 `IValidator<UserViewModel>`：</font>

```csharp
// The long way - 冗长的方式
builder.Bind<IValidator<UserViewModel>>().To<UserViewModelValidator>();
```

However, if you have many validators, you'll need many lines of configuration. It's better to tell StyletIoC to discover all `IValidator<T>` implementations, and bind them itself, by this doing:

---
><font color="#63aebb" face="微软雅黑">如果你有许多验证器，则需要多行配置。最好告诉 StyletIoC 所有 IValidator<T> 实现，并通过以下方式绑定它们：</font>

```csharp
// The short way - 简短的方式
builder.Bind(typeof(IValidator<>)).ToAllImplementations();
```

There we go! When StyletIoC tries to create a new `FluentValidationAdapter<UserViewModel>`, it will see that it needs an `IValidator<UserViewModel>`, and will instantiate a new `UserViewModelValidator`.

当  StyletIoC 尝试创建新的 `FluentValidationAdapter<UserViewModel>`，会注意到需要 `IValidator<UserViewModel>`，并将实例化新的 `UserViewModelValidator`。

Now you can see why we use `IModelValidator<T>` instead of `IModelValidator` here. Had `UserViewModel` required an `IModelValidator`, StyletIoC wouldn't have been able to work out that it should create a `FluentValidationAdapter<UserViewModel>`, rather than, say, a `FluentValidationAdapter<LogInViewModel>`. By adding type information to `IModelValidator`, we give the IoC container enough information to work with.

现在你可以看到为什么我们在这里使用 `IModelValidator<T>` 而不是 `IModelValidator`。如果 `UserViewModel`需要 `IModelValidator`，那么 StyletIoC 无法确定它应该创建 `FluentValidationAdapter<UserViewModel>`，而不是 `FluentValidationAdapter<LogInViewModel>`。通过向 `IModelValidator` 添加类型信息，为 IoC 容器提供足够的信息来使用。


Using a pre-made IModelValidator - 使用预先编写的 IModelValidator
--------------------------------

I've written the following IModelValidator implementations, which you're welcome to use:

---
><font color="#63aebb" face="微软雅黑">我编写了以下 IModelValidator 实现，欢迎使用：</font>

1. [FluentValidationAdapter](./FluentValidationAdapter.md)

If you write one, and you're happy to share it, please let me know and I'll add it.

---
><font color="#63aebb" face="微软雅黑">如果你也写了一个并乐意分享它,告诉我,我添加它。</font>


Implementing IModelValidator (Synchronously) - 实现 IModelValidator (同步)
--------------------------------------------

Writing an `IModelValidator` implementation is conceptually straightforward, but has a few gotchas. As before, this section will assume we're implementing an adapter for the FluentValidation library, although you can apply the knowledge gained here to write an adapter for almost any library.

For now, let's off assume that all of our validations are synchronous. For the methods which return Tasks, we'll just return a completed task. Easy.

First off, we'll implement `IModelValidator<T>` for reasons discussed in the previous section. It will also need to accept a `IValidator<T>`, as a constructor argument, like so:

---
><font color="#63aebb" face="微软雅黑">编写 `IModelValidator` 实现在概念上很简单，但是有一些问题。与前面一样，本节将假设我们正在为 FluentValidation 库实现适配器，你可以应用这里获得的知识为几乎所有库编写适配器。

>让我们假设我们所有的验证都是同步的。对于返回 Tasks 的方法，我们只返回一个已完成的任务。

>我们实现 `IModelValidator<T>` ，原因在前一节中讨论过。它还需要接受 `IValidator<T>` 作为构造函数参数，如下:</font>

```csharp
public class FluentValidationAdapter : IModelValidator<T>
{
   private readonly IValidator<T> validator;
   public FluentValidationAdapter(IValidator<T> validator)
   {
      this.validator = validator;
   }
}
```

Remember that `ValidatingModelBase` wants an `IModelValidator` which is specialised to validating a particular ViewModel instance, as it adds more flexibility. This means that `ValidationModelBase` can call `ValidateAllPropertiesAsync()`, and the correct ViewModel instance will be validated. However, here we have a chicken-and-egg situation - in order to specialise the adapter, the ViewModel must exist. However, the ViewModel can't be instantiated until after the adapter has been validated, as the ViewModel requires the adapter as a constructor argument.

The solution is the `Initialize(object subject)` method. This is called by `ValidatingModelBase` when it's passed a new adapter, and it will pass itself as the argument. The adapter will then store this instance, and use it when it's running validations. Like this:

---
><font color="#63aebb" face="微软雅黑">记住，`ValidatingModelBase` 需要 `IModelValidator`，它专门用于验证特定的 ViewModel 实例，因为它增加了更多的灵活性。这意味着 `ValidationModelBase` 可以调用 `ValidateAllPropertiesAsync()`，并将验证 ViewModel 实例是否正确。然而，这里我们有一个先有鸡还是先有蛋的问题 — 为了限定适配器，ViewModel 必须存在。但是，在适配器被验证之后，才能实例化 ViewModel，因为 ViewModel 需要适配器作为构造函数参数。

>解决方法是 `Initialize(object subject)`。`ValidatingModelBase` 在传递新适配器时调用的，它将自己作为参数传递。适配器将存储此实例，并在运行验证时使用它。像这样：</font>

```csharp
public class FluentValidationAdapter : IModelValidator<T>
{
   private readonly IValidator<T> validator;
   private T subject;

   public FluentValidationAdapter(IValidator<T> validator)
   {
      this.validator = validator;
   }

   public void Initialize(object subject)
   {
      this.subject = (T)subject;
   }
}
```

Now, implementing `ValidatePropertyAsync`. This should validate a single property, and return a list of validation errors, or null/emptyarray if there are none. Using FluentValidation to perform synchronous validation, this might look like this:

---
><font color="#63aebb" face="微软雅黑">实现 `ValidatePropertyAsync`。这应该验证单个属性，并返回验证错误列表，如果没有，则返回null/emptyarray。使用 FluentValidation 来执行同步验证，如下:</font>

```csharp
public Task<IEnumerable<string>> ValidatePropertyAsync(string propertyName)
{
   var errors = this.validator.Validate(this.subject, propertyName).Errors.Select(x => x.ErrorMessage);
   return Task.FromResult(errors);
}
```

Similarly, the `ValidateAllPropertiesAsync` method validates all properties, and returns a Dictionary of `{ propertyName => array of validation errors }`. If a property doesn't have any validation errors, you can either omit it from the Dictionary entirely, or have its value set to null/emptyarray.

---
><font color="#63aebb" face="微软雅黑">同样，`ValidateAllPropertiesAsync` 方法验证所有属性，并返回 `{propertyName =>验证错误数组}` 字典。如果属性没有任何验证错误，你可以从字典中完全删除它，或者将其值设置为null/emptyarray。</font>

```csharp
public Task<Dictionary<string, IEnumerable<string>>> ValidateAllPropertiesAsync()
{
   var errors = this.validator.Validate(this.subject).Errors.GroupBy(x => x.PropertyName).ToDictionary(x => x.Key, x => x.Select(failure => failure.ErrorMessage));
   return Task.FromResult(errors);
}
```

Put that all together, and you have your adapter!

---
><font color="#63aebb" face="微软雅黑">把这些放在一起，你就有了你的适配器！</font>

Implementing IModelValidator (Asynchronously) - 实现IModelValidator(异步)
---------------------------------------------

Implementing asynchronous validation (for libraries which support it is a bit trickier).

First, remember that `ValidatingModelBase` has both a set of synchronous methods (`Validate`, `ValidateProperty`) and asynchronous methods (`ValidateAsync`, `ValidatePropertyAsync`). Under the hood, the synchronous versions call the asynchronous versions, but block the thread until the asynchronous operation has completed (using `Task.Wait()`).

Now, if you're used tasks much, this should be setting off alarm bells. You see, when you `await DoSomethingAsync(); DoSomethingElse();`, you're saying "*capture the current thread [***]. when the `DoSomethingAsync()` asynchronous operation completes, I want you post a message to that captured thread, telling it to run `DoSomethingElse()`*". However, if that thread is waiting for the asynchronous operation to complete, it will never receive that message, the operation will never complete, and you have deadlock.

[\*] not entirely true - it captures the current `SynchronizationContext`. On the UI thread, this amounts to the same, though.

Put another way, this means that the following code, run from the UI thread, will deadlock:

---
><font color="#63aebb" face="微软雅黑">实现异步验证(对于支持它的库来说要复杂一些)。

>首先，请记住 `ValidatingModelBase` 有一组同步方法(`Validate`，`ValidateProperty`)和异步方法(`ValidateAsync`，`ValidatePropertyAsync`)。在底层，同步版本调用异步版本，但是阻塞线程直到异步操作完成(使用 `Task.Wait()`)。

>如果你使用多任务，这应该注意。当 `await DoSomethingAsync(); DoSomethingElse();`，你说："*捕获当前线程 [***]。当 `DoSomethingAsync()` 异步操作完成时，我希望你向捕获的线程发送一条消息，告诉它运行 `DoSomethingElse()`*"。但是，如果该线程正在等待异步操作完成，它将永远不会收到该消息，操作将永远不会完成，并且出现死锁。

>[\*]不完全正确 - 它捕获了当前的 `SynchronizationContext`。在 UI 线程上，这是相同的。

>换句话说，这意味着从UI线程运行的以下代码将会死锁：</font>

```csharp
public async Task DoSomethingAsync()
{
   await Task.Delay(100);
}

// ... 
DoSomethingAsync().Wait();
DoSomethingElse();
```

When the `Task.Delay(100)` task completes, it will post a message back to the UI thread saying "right, run `DoSomethingElse()`". However, the UI thread is stuck on that `Wait()`, will never process the message, and you're deadlocked.

Why is this relevant? Well. If you write an `IModelValidator<T>` method which looks like this:

---
><font color="#63aebb" face="微软雅黑">当 `Task.Delay(100)` 任务完成时，它会发布一个消息给 UI 线程说：“好了，执行 `DoSomethingElse()`”。但是，UI 线程卡在 `Wait()` 上，永远不会处理消息，而且你已陷入僵局。

>为什么这有关系？如果你编写如下 `IModelValidator<T>` 方法：</font>

```csharp
public async Task<IEnumerable<string>> ValidatePropertyAsync(string propertyName)
{
   var result = await this.Validator.ValidateAsync(this.subject, propertyName);
   return result.Errors.Select(x => x.ErrorMessage);
}
```

Then call `ValidateProperty`, **you will deadlock**.

The trick is to tell `await` not to capture the current thread, using `ConfigureAwait(false)`, i.e.

---
><font color="#63aebb" face="微软雅黑">然后调用 `ValidateProperty`， **你会死锁**。

>诀窍是告诉 `await` 不要捕获当前线程，使用 `ConfigureAwait(false)`，例如。</font>

```csharp
public async Task<IEnumerable<string>> ValidatePropertyAsync(string propertyName)
{
   var result = await this.Validator.ValidateAsync(this.subject, propertyName).ConfigureAwait(false);
   return result.Errors.Select(x => x.ErrorMessage);
}
```

Now, the `return result.Errors...` line will be run on another thread (rather than posted to the UI thread), and no deadlock occurs.

---
><font color="#63aebb" face="微软雅黑">现在，`result.Errors.Select(x => x.ErrorMessage);` 将在另一个线程上运行(而不是发布到 UI 线程上)，并且不会发生死锁。</font>

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[StyletIoC-Introduction - StyletIoC 简介](./Ioc/StyletIoC-Introduction.md)