This page contains various other bits and bobs which deserve a mention, but aren't large enough to deserve their own page.

---
><font color="#63aebb" face="微软雅黑">这个页面包含一此零散的内容，它们值得一提，但是不够大，不值得使用单独的页面。</font>

Circular Dependencies - 循环依赖
---------------------

Circular dependencies, except those of the type documented below, cause a StackOverflow exception. It isn't trivial to spot these ahead of time, and while a StackOverflow exception isn't ideal, it's not worth the complexity of avoiding it.

---
><font color="#63aebb" face="微软雅黑">循环依赖（下面记录的类型除外）会导致 StackOverflow 异常。提前发现这些并非易事，虽然 StackOverflow 异常并不理想，但避免它的复杂性并不值得。</font>

Parent/Child Circular Dependencies - 父 / 子循环依赖
----------------------------------

Say you have something like this - 假设:

```csharp
class Parent
{
   public Parent(Child child) { ... }
}

class Child
{
   public Parent Parent { get; set; }
}
```

where you want StyletIoC to be able to create an instance of Parent, or Child, and have the other created appropriately.

The trick is to use factories to create both, and not to use the container to resolve an instance of Child and creating a Parent, and vice versa. It's a bit messy, but then circular dependencies are messy anyway.

Child's Parent property must *not* have an `[Inject]` attribute on it, or the `BuildUp` step will cause a StackOverflow.

---
><font color="#63aebb" face="微软雅黑">你希望 StyletIoC 能够创建父实例或子实例，并适当地创建另一个实例。

>技巧是使用工厂来创建两者，而不是使用容器来解析 Child 实例并创建 Parent 实例，反之亦然。它有点乱，但无论如何循环依赖都很混乱。

>Child 的 Parent 属性不能包含 `[Inject]` 修饰属性，否则该 `BuildUp` 步骤将导致 StackOverflow 异常。</font>

```csharp
builder.Bind<Parent>().ToFactory(container =>
{
   var child = new Child();
   container.BuildUp(child); // If child requires any property injection - 如果 Child 需要属性注入
   var parent = new Parent(child);
   child.Parent = parent;
   return parent; // The parent will be automatically built up by StyletIoC - 由 StyletIoC 自动构建 Parent
});
builder.Bind<Child>().ToFactory(container =>
{
   var child = new Child();
   var parent = new Parent(child);
   container.BuildUp(parent); // If parent requires any property injection - 如果 Parent 需要属性注入
   child.Parent = parent;
   return child; // The child will be automatically built up by StyletIoC - 由 StyletIoC 自动构建 Child
});
```

[目录](./../Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[StyletIoC-Performance - StyletIoC 性能](./StyletIoC-Performance.md)