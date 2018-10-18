
In almost every project I've needed to hide/show an element based on some bool value in the ViewModel. You can do this with DataTriggers, or using a converter.

The converter implementation is very simple: when bound to a bool property, if it reads a true value, it returns one (preconfigured) visibility, if it reads a false value, it returns another.

If you bind in to a property of a type other than bool, it will use the following rules:

1. If the value is null, it's treated as false
2. If the value is 0 (as an int, float, double, etc), it's treated as false
3. If the value is an empty collection, dictionary, etc, it's treated as false
4. Otherwise, it's treated as true

This matches the truthsy/falsy rules seen in many languages. It's also handy if, say, you want to show a `ListView` if and only if the collection it's bound to isn't empty.

Basic example usage:

---
><font color="#63aebb" face="微软雅黑">在几乎每个项目中，我都需要根据 ViewModel 中的一些 bool 值来 `隐藏` / `显示` 元素。你可以使用 DataTriggers 或使用转换器执行此操作。

>转换器实现非常简单：当绑定到bool属性时，如果它读取true值，则返回一个（预配置的）可见性，如果它读取false值，则返回另一个。

>如果绑定到 bool 值以外的类型的属性，它将使用以下规则：

>1. 如果值为 null，则为 false
>2. 如果值为0（int，float，double等），则为 false
>3. 如果值是空集合，字典等，则为 false
>4. 否则为 true

>这符合在许多语言中看到的 `真` / `假` 规则。如果 `ListView` 的绑定集合不是空时才显示，用它很方便。</font>

```xml
<!-- In any .Resources section - doesn't have to be Window.Resources -->
<Window.Resources>
   <s:BoolToVisibilityConverter x:Key="boolToVisConverter" TrueVisibility="Visible" FalseVisibility="Hidden"/>
</Window.Resources>

<!-- Later in code -->
<TextBlock Visibility="{Binding SomeBoolProperty, Converter={StaticResource boolToVisConverter}}"/>
```

If you want the (usual) case of a converter which uses `Visiblity.Visible` for the true case, and `Visibility.Collapsed` for the false case, there's a shortcut:

如果你想要(通常的)转换器用 `Visiblity.Visible` 表示 `true`，用 `Visibility.Collapsed` 表示 `false`，有一种快捷的方法:

```xml
<TextBlock Visibility="{Binding SomeBoolProperty, Converter={x:Static s:BoolToVisibilityConverter.Instance}}"/>
```

Likewise, if you want the (slightly less usual) case of `Visibility.Collapsed` for the true case and `Visibility.Visible` for the false case, there's a similar shortcut:

同样，如果你想 `Visibility.Collapsed` 表示 `true` 和 `Visiblity.Visible` 表示 `false`，也有一种快捷方法：


```xml
<TextBlock Visibility="{Binding SomeBoolProperty, Converter={x:Static s:BoolToVisibilityConverter.InverseInstance}}"/>
```

[目录](./Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[IoC: Static Service Locator - 静态服务定位](./IoC-Static-Service-Locator.md)