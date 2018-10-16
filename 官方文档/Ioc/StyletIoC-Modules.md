Any application with a reasonable amount of complexity will include a lot of IoC container configuration: there are lots of explicit binding of interfaces to implementations, of specifying singletons, and other (completely necessary) configuraiton.

The default approach is to put all of this configuration in your bootloader, and this makes sense: there's one single place where all of the configuration happens, and it's easy to see what's going on.

However, as your design grows, you might find yourself looking for a way to split up the IoC container configuration a bit. This is the role of modules.

I think an example is the best way of demonstrating how these work. I'm not going to include hundreds of lines of IoC configuration just for the sake of an example - I'll only include a few, and let you imagine the rest.

Before:

```csharp
class Bootstrapper : Bootstrapper<IShellViewModel>
{
   protected override void ConfigureIoC(IStyletIoCBuilder builder)
   {
      builder.Bind<IServiceAComponent1>().To<ServiceAComponent1>().InSingletonScope();
      builder.Bind<IServiceAComponent2>().To<ServiceAComponent2>().InSingletonScope();
      builder.Bind<ISomeFactory>().ToAbstractFactory();
   }
}
```

After:

```csharp
class ServiceAModule : StyletIoCModule
{
   protected override void Load()
   {
      Bind<IServiceAComponent1>().To<ServiceAComponent1>().InSingletonScope();
      Bind<IServiceAComponent2>().To<ServiceAComponent2>().InSingletonScope();
   }
}

class FactoriesModule : StyletIoCModule
{
   protected override void Load()
   {
      Bind<ISomeFactory>().ToAbstractFactory();
   }
}

class Bootstrapper : Bootstrapper<IShellViewModel>
{
   protected override void ConfigureIoC(IStyletIoCBuilder builder)
   {
      builder.AddModule(new ServiceAModule());
      builder.AddModule(new FactoriesModulie());
   }
}
```

As you can see, we've been able to break up the container configuration into multiple modules. Each module is responsible for doing a bit of container configuration, and then the bootstrapper adds all of these modules to the container. While there is a bit more code, you've got a tool to separate and modularise things a bit, which is sometimes very useful.

Modules are simply classes which inherit from the abstract `StyletIoCModule`. There is one abstract method you must override `void Load()`: this is called when the module is added to the container, and is responsible for adding all of your bindings. If you try and add bindings anywhere else (for example in the constructor), they won't have any effect.

There are two protected methods available on `StyeltIoCModule` for you to use: `Bind<TService>()` and `Bind(Type serviceType)`. These work exactly the same as the methods on `IStyletIoCBuilder`, and you use these to create your bindings.

Finally, there's a method on `IStyletIoCBuilder` called `AddModule`, which takes a module instance as an argument. This simply runs the module's `Load` method, and adds all of the resulting bindings to itself. If you add the same module twice, you'll get duplicating bindings - simple.