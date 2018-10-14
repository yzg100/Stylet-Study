The ViewManager is one of Stylet's core components, and is responsible for taking a ViewModel, and locating the correct View for it. It then instantiates that View, and binds it to the ViewModel.

This article will teach you how this process works, and how to modify it.

Locating a View
---------------

The default view-location strategy is pretty simple: replace the string "ViewModel" with the string "View" in the type and namespace of the ViewModel, then search all configured assemblies for a type with that name.

That's it.

'Configured Assemblies' means all assemblies in the ViewAssemblies property of the ViewManager. If you want to search more assemblies, add them to this list (see further down for an example).

This process is controlled by ViewManager.LocateViewForModel, so if you want to modify it, go ahead and subclass `ViewManager` - see the section at the bottom of this article.


Creating a View
---------------

Once the view type has been identified, it's instantiated by calling a factory method supplied to the ViewManager. The bootstrapper sets this up to call into your IoC container by default. `InitializeComponent` is then called, which means you can delete your codebehind if you want.

This is handled by `ViewManager.CreateViewForModel`, if you fancy overriding it.


Binding the View to its ViewModel
---------------------------------

Binding a ViewModel and its corresponding View is pretty straightforward:
 - The View's DataContext is set to the ViewModel.
 - The View's ActionTarget is set to the ViewModel (see [[Actions]]).
 - If the ViewModel implements `IViewAware`, its `AttachView` method is called, and the View passed (see [[Screens and Conductors]]).


Dealing with `View.Model` Changes
---------------------------------

The ViewManager is also responsible for reacting to changes of a `ContentControl`'s `View.Model` attached property - see [[ViewModel First]].

It will grab the new value of the attached property (which should have been bound to a ViewModel instance), and use `CreateViewForModel` and `BindViewToModel` to create a new configured View instance, before setting it as the `ContentControl`'s content.

Configuring the ViewManager
---------------------------

The ViewManager is configurable. To configure it, grab an instance of it in your `Configure` method:

```csharp
protected override void Configure()
{
    var viewManager = this.Container.Get<ViewManager>();
    // ...
}
```

Once you've got it, there are several things you can configure on it:

 - `ViewSuffix`: the suffix of your View class names. Defaults to "View".
 - `ViewModelSuffix`: the suffix of your View class names. Defaults to "ViewModel".
 - `ViewAssemblies`: which assemblies to scan for views. Defaults to the assembly containing your bootstrapper.
 - `NamespaceTransformations`: this is a dictionary of `from -> to` substitutions, and allows your Views and ViewModels to live in different namespaces. If your Views live in "Foo.Frontend.Views" and your ViewModels live in "Foo.FrontendLogic.ViewModels", you could add an entry to this dictionary with the key "Foo.Frontend" and the value "Foo.FrontendLogic". Note that you have to include all root namespaces: you couldn't have used a transformation from "Frontend" to "FrontendLogic" here.


Creating your own ViewManager
-----------------------------

When Stylet needs the ViewManager, it retrieves an instance of `IViewManager` from the IoC container. This means that you can create an implementation of IViewManager you want and register it with the IoC container, and Stylet will pick it up and use it.

A very simplistic example:

```csharp
// CustomViewManager.cs

public class CustomViewManager : ViewManager
{
   public CustomViewManager(ViewManagerConfig config)
      : base(config)
   { }

   public override UIElement CreateAndSetupViewForModel(object model)
   {
      // This dummy application only has one view, and it's this one :)
      return new SingletonView();
   }
}

// Bootstrapper.cs

public class Bootstrapper : Bootstrapper<MyRootViewModel>
{
   protected override void ConfigureIoC(IStyletIoCBuilder builder)
   {
      builder.Bind<IViewManager>().To<CustomViewManager>();
   }
}
```

And that's it!

A more complete example can be found in the [`Stylet.Samples.OverridingViewManager` sample](https://github.com/canton7/Stylet/tree/master/Samples/Stylet.Samples.OverridingViewManager). 