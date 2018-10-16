This page goes into more detail on Constructor Injection and Property Injection.

Constructor Injection
---------------------

Constructor injection happens automatically when StyletIoC creates a new instance of a type, provided that the type was bound using the `Bind<...>().To<...>()` syntax, and not `Bind<...>().ToFactory(...)`.

When presented with a type to construct, StyletIoC first needs to determine which constructor to call, if there's more than one:

1. If a constructor is decorated with the `[Inject]` attribute, StyletIoC will use that.
2. Any constructors which contain parameters with types which StyletIoC doesn't know how to resolve, and which don't have a default value, are discarded.
2. The constructor with the most parameters which StyletIoC can provide values for is selected, and default values used for parameters which StyletIoC can't provide values for.

If a constructor type is an `IEnumerable<T>` for some `T`, and you haven't explicitly registered that `IEnumerable<T>` as a service with StyletIoC, but you *did* register one or more types which implement `T`, StyletIoC will construct an `IEnumerable<T>` containing instances of those types.

Property Injection
------------------

After constructing a type, StyletIoC will inject values into all properties and fields decorated with the `[Inject]` attribute. It will inject into both public, protected and private properties and fields - it's not recommended to inject values into private members (since it becomes impossible to set up the type without reflection), but you may need it.

The main problem with property injection is that you properties should really be world-writable (so it can be set up without reflection, for example in a unit test), even if they shouldn't be from an encapsulation point of view. This often ends up with properties which are `{ set; private get; }`, which is really horrible. For this reason, constructor injection is almost always preferred.

Because the properties are injected after the type is constructed, everything which happens in the constructor must not rely on these properties being populated. This is another reason to prefer constructor injection. If you *do* need to know when the properties are injected, implement the interface `IInjectionAware`. The interface method `PropertiesInjected` will be called when all properties have been injected.

Because property injection requires properties to be decorared with `[Inject]`, your type can't be IoC-container-agnostic, as they can with constructor injection. This will bite you if you want to change IoC container.

As with constructor injection, a property of type `IEnumerable<T>` for some `T` will have a collection of that type injected.

You can also perform property injection if StyletIoC did not construct the type - call `IContainer.BuildUp`. For example:

```csharp
var car = new OldBanger():
ioc.BuildUp(car);
```

Examples:

```csharp
class OldBanger
{
   // Will not be injected - no [Inject] attribute
   public IEngine Engine { get; set; }
}

class OldBanger
{
   // Will be injected, provided a type is registered for the service IEngine
   [Inject]
   public IEngine Engine { get; set; }
}

class Garage
{
   // Will inject all types registered for the service IVehicle
   [Inject]
   public IEnumerable<IVehicle> Vehicles { get; set; }
}

class OldBanger
{
   // Fields are also injected into
   [Inject]
   public IEngine Engine;
}

class OldBanger
{
   // This works true, although you really shouldn't do it
   [Inject]
   private IEngine Engine;
}

class OldBanger : IInjectionAware
{
   [Inject]
   public IEngine Engine { get; set; }

   public PropertiesInjected()
   {
      // Do something with this.Engine
   }
}
```