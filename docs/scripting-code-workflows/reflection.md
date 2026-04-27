---
title: Type Library & Reflection
icon: "🌲"
sources:
  - engine/Sandbox.Reflection
created: 2026-04-27
updated: 2026-04-27
---

# Type Library & Reflection

The `TypeLibrary` is a highly optimized reflection caching system that tracks C# classes and properties across all loaded assemblies to power the Editor, UI, and ActionGraph.

## Quick Start Example

```csharp
using Sandbox;

public sealed class ReflectionExample : Component
{
    protected override void OnStart()
    {
        var myTypes = TypeLibrary.GetTypes<Component>()
            .Where( x => x.HasAttribute<CategoryAttribute>() );

        foreach ( var typeDesc in myTypes )
        {
            Log.Info( $"Found custom component: {typeDesc.Name}" );
        }
    }
}
```

## Common Patterns

### Finding Specific Types
The most common use of `TypeLibrary` is querying for types that implement a certain base class or interface, and optionally filtering them by attributes.

```csharp
using Sandbox;

public sealed class ReflectionExample : Component
{
    protected override void OnStart()
    {
        // Get all types that inherit from 'Component'
        var myTypes = TypeLibrary.GetTypes<Component>()
            .Where( x => x.HasAttribute<CategoryAttribute>() );

        foreach ( var typeDesc in myTypes )
        {
            Log.Info( $"Found custom component: {typeDesc.Name}" );
            
            // You can also instantiate the type directly from the description:
            var instance = typeDesc.Create<Component>();
        }
    }
}
```

### Retrieving Types by Name
If you need to resolve a type by its string name (useful for serialization or data-driven loading):

```csharp
// Get a TypeDescription by its class name
TypeDescription specificType = TypeLibrary.GetType( "MyCustomComponent" );

// Or get a TypeDescription that inherits from a specific base type
TypeDescription specificComponent = TypeLibrary.GetType<Component>( "MyCustomComponent" );
```

### Checking Attributes Rapidly
`TypeLibrary` heavily caches attribute lookups, making it very fast to check if a type has a specific attribute.

```csharp
bool isCategory = TypeLibrary.HasAttribute<CategoryAttribute>( typeof( MyCustomComponent ) );
```

### SerializedObjects
The Editor makes heavy use of `TypeLibrary.GetSerializedObject( object )` to create a `SerializedObject` representation of a C# class instance. This abstraction is what the Inspector panels read and write to, allowing properties to be displayed and edited natively in the Editor UI without the Inspector needing to know the hardcoded C# type.

## Performance Edge Cases

While `TypeLibrary` is much faster than standard `System.Reflection`, you should still avoid querying it every frame in `OnUpdate()`. `TypeLibrary.GetTypes<T>()` returns a dynamically built enumerable; performing LINQ queries against it repeatedly will cause unnecessary garbage collection (GC) allocations.

**Best Practice:** Query `TypeLibrary` once in `OnStart` or `OnAwake`, cache the resulting list of `TypeDescription` objects into a member field, and reuse that list throughout the component's lifetime.
