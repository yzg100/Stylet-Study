Sometimes you want to display some object to the user, but you want to associate a custom (string) label with it, which will be displayed by your View. So you create a quick class to wrap up your object and attach this label.

Then you'll want to override ToString so that your View displays just the label, and Equals and GetHashCode so they work properly with things that have a `SelectedItem` of some sort (e.g. `ComboBox`). Finally, you'll want to implement `INotifyPropertyChanged` so that changes to this can be picked up by the View.

This is all that `LabelledValue<T>` is - a class with a string `Label` property and a `T` `Value` property. Plus an overridden `ToString`, `GetHashCode`, `Equals`, and implementing `INotifyPropertyChanged`.

For example:

---
><font color="#63aebb" face="微软雅黑">有时你希望向用户显示某个对象，但你希望将自定义（字符串）标签与其关联，该标签将由你的View显示。因此，你创建一个快速类来包装对象并附加此标签。

>然后，你要重写 ToString，让 View 只显示标签，以及 Equals 和 GetHashCode，让它们可以正常处理具有 `SelectedItem` 之类的对象（例如 `ComboBox`）。最后，你需要实现，`INotifyPropertyChanged` 以便 View 可以对此进行更改。

>这就是 `LabelledValue<T>` - 具有字符串 `Label` 属性和 `T` `Value` 属性的类。加上重写 `ToString`, `GetHashCode`, `Equals`，和实现 `INotifyPropertyChanged` 接口。

>例如：</font>

```csharp
public enum MyEnum
{
   Foo,
   Bar,
   Baz
}

class MyViewModel
{
   // Implement INotifyPropertyChanged if you want
   public BindableCollection<LabelledValue<MyEnum>> EnumValues { get; private set; }
   public LabelledValue<MyEnum> SelectedEnumValue { get; set; }

   public MyViewModel()
   {
      this.EnumValues = new BindableCollection<LabelledValue<MyEnum>>()
      {
         LabelledValue.Create("Foo Value", MyEnum.Foo),
         LabelledValue.Create("Bar Value", MyEnum.Bar),
         LabelledValue.Create("Baz Value", MyEnum.Baz),
      };

      this.SelectedEnumValue = this.EnumValues[0];
   }
}
```

Then your view...

```xml
<ComboBox ItemsSource="{Binding EnumValues}" SelectedItem="{Binding SelectedEnumValue}"/>
```

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[DebugConverter](./DebugConverter.md)