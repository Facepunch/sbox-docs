---
title: "Code Generation"
icon: "🧞"
created: 2023-11-19
updated: 2026-04-27
sources:
  - engine/Sandbox.System/CodeGen/CodeGeneratorAttribute.cs
  - engine/Sandbox.System/CodeGen/CodeGeneratorFlags.cs
  - engine/Sandbox.System/CodeGen/WrappedMethod.cs
---

# Code Generation

Learn how to use `[CodeGenerator]` to wrap methods and properties at compile-time to inject custom behavior, intercept calls, or build your own attributes like RPCs or networked variables.

## Quick Working Example

```csharp
using Sandbox;
using System;

// 1. Create a custom attribute and decorate it with [CodeGenerator]
[AttributeUsage( AttributeTargets.Method )]
[CodeGenerator( CodeGeneratorFlags.WrapMethod | CodeGeneratorFlags.Instance, "OnLogMethodInvoked" )]
public class LogMethodAttribute : Attribute { }

public class MyObject
{
	// 2. Use your custom attribute on a method
	[LogMethod]
	public void DoSomething( string message )
	{
		Log.Info( $"Original method ran with: {message}" );
	}
	
	// 3. Define the callback method that the CodeGenerator will point to
	internal void OnLogMethodInvoked( WrappedMethod m, string message )
	{
		Log.Info( $"Intercepted method call for {m.MethodName}!" );
		
		// Call the original method
		m.Resume();
		
		Log.Info( "Finished intercepted method call!" );
	}
}
```

### Wrapping Methods

When you use `CodeGeneratorFlags.WrapMethod`, your callback method should accept a `WrappedMethod` struct as its first parameter, followed by the original method's arguments. 

```csharp
[AttributeUsage( AttributeTargets.Method )]
[CodeGenerator( CodeGeneratorFlags.WrapMethod | CodeGeneratorFlags.Instance, "OnMethodInvoked" )]
public class WrapCallAttribute : Attribute { }

public class MethodWrapper
{
	[WrapCall]
	public void MyMethod( bool enabled ) { }

	[WrapCall]
	public void MyOtherMethod( string text, int count ) { }

	// Overload for MyMethod
	internal void OnMethodInvoked( WrappedMethod m, bool enabled ) 
	{
		m.Resume();
	}
	
	// Overload for MyOtherMethod
	internal void OnMethodInvoked( WrappedMethod m, string text, int count ) 
	{
		m.Resume();
	}
}
```

If your wrapped method returns a value, the callback must return that type, and accept a `WrappedMethod<T>`.

```csharp
internal int OnMethodInvoked( WrappedMethod<int> m )
{
	return m.Resume();
}
```

### Wrapping Properties

You can wrap properties to intercept when they are read or written by using `CodeGeneratorFlags.WrapPropertyGet` and `CodeGeneratorFlags.WrapPropertySet`.

```csharp
[AttributeUsage( AttributeTargets.Property )]
[CodeGenerator( CodeGeneratorFlags.WrapPropertySet | CodeGeneratorFlags.Instance, "OnWrapSet" )]
[CodeGenerator( CodeGeneratorFlags.WrapPropertyGet | CodeGeneratorFlags.Instance, "OnWrapGet" )]
public class WrapGetSetAttribute : Attribute { }

public class PropertyWrapper
{
	[WrapGetSet] 
	public string MyString { get; set; }
	
	// Callback for Getters
	internal T OnWrapGet<T>( WrappedPropertyGet<T> p )
	{
		// You can return p.Value to return the original backing field's value, 
		// or return a completely different value of type T.
		return p.Value;
	}

	// Callback for Setters
	internal void OnWrapSet<T>( WrappedPropertySet<T> p )
	{
		// You can execute code here before the property is set.
		
		// Use p.Setter() to apply the change to the backing field.
		p.Setter( p.Value );
	}
}
```

### Static vs Instance

You must specify whether your code generation targets static or instance methods (or both). If your callback is handling a static method or property, the callback itself **must** be static.

```csharp
[CodeGenerator( CodeGeneratorFlags.WrapMethod | CodeGeneratorFlags.Static, "MyStaticClass.OnWrapStatic" )]
public class WrapStaticAttribute : Attribute { }

public static class MyStaticClass
{
	// Notice this callback is static
	internal static void OnWrapStatic( WrappedMethod m )
	{
		m.Resume();
	}
}
```

The `CodeGeneratorFlags` enum defines what parts of the code you want to wrap. You can combine these using bitwise OR (`|`).

| Flag | Description |
|---|---|
| `WrapPropertyGet` | Wrap the get accessor of a property. |
| `WrapPropertySet` | Wrap the set accessor of a property. |
| `WrapMethod` | Wrap a method call. |
| `Static` | Apply this to a static property or method. |
| `Instance` | Apply this to an instance property or method. |

The `[CodeGenerator]` attribute takes the following parameters:

| Parameter | Description |
|---|---|
| `Type` | The `CodeGeneratorFlags` defining what to wrap. |
| `CallbackName` | The exact string name of the method to call. If calling a static method on another class, use the fully qualified name (e.g. `"MyClass.MyCallback"`). |
| `Priority` | (Optional) Attributes with a higher priority will wrap the target first. Default is `0`. |

## Realistic Examples

s&box's built-in `[Rpc.Broadcast]` and `[Sync]` are themselves implemented with `[CodeGenerator]`. You don't need to roll your own — but seeing how they're built is the clearest way to understand what `[CodeGenerator]` is for.

### A custom RPC attribute

```csharp
[CodeGenerator( CodeGeneratorFlags.WrapMethod | CodeGeneratorFlags.Instance, "OnRpcInvoked" )]
public class MyRpcAttribute : Attribute { }

public class MyObject
{
    [MyRpc]
    public void SendMessage( string message )
    {
        Log.Info( message );
    }

    internal void OnRpcInvoked( WrappedMethod m, params object[] args )
    {
        if ( IsServer )
        {
            // Send a networked message with the args + method name to all clients,
            // then resume so it also runs locally on the server.
        }
        m.Resume();
    }
}
```

### A custom networked-variable attribute

Combine a property-get and property-set wrap to build something like `[Sync]`:

```csharp
[CodeGenerator( CodeGeneratorFlags.WrapPropertyGet | CodeGeneratorFlags.Instance, "OnNetVarGet" )]
[CodeGenerator( CodeGeneratorFlags.WrapPropertySet | CodeGeneratorFlags.Instance, "OnNetVarSet" )]
public class MyNetVarAttribute : Attribute { }

public class MyObject
{
    [MyNetVar] public string Name { get; set; }

    internal T OnNetVarGet<T>( WrappedPropertyGet<T> p )
    {
        // Pull the latest value from your replication table if there is one
        if ( MyNetVarTable.TryGetValue( p.PropertyName, out var netValue ) )
            return (T)netValue;

        return p.Value;
    }

    internal void OnNetVarSet<T>( WrappedPropertySet<T> p )
    {
        if ( IsServer )
        {
            MyNetVarTable[p.PropertyName] = p.Value;
            // Broadcast the change to clients
        }
        p.Setter( p.Value );
    }
}
```

### Wrapping everything at once

You can stack flags to wrap methods and both property accessors with a single attribute. Note that mixing `Static` requires a static callback (use a fully-qualified name).

```csharp
[CodeGenerator(
    CodeGeneratorFlags.WrapMethod |
    CodeGeneratorFlags.WrapPropertyGet | CodeGeneratorFlags.WrapPropertySet |
    CodeGeneratorFlags.Static | CodeGeneratorFlags.Instance,
    "MyStaticHandler.OnWrapAnything" )]
public class WrapAnythingAttribute : Attribute { }

public static class MyStaticHandler
{
    internal static void OnWrapAnything( WrappedMethod m, params object[] args ) => m.Resume();
    internal static T OnWrapAnything<T>( WrappedMethod<T> m, params object[] args ) => m.Resume();
    internal static T OnWrapAnything<T>( WrappedPropertyGet<T> p ) => p.Value;
    internal static void OnWrapAnything<T>( WrappedPropertySet<T> p ) => p.Setter( p.Value );
}
```

## Troubleshooting

:::warning Forgot to call Resume / Setter
If you wrap a method or property setter and notice that the original code is no longer running or the property is no longer saving its value, you likely forgot to call `m.Resume();` or `p.Setter( p.Value );` inside your callback!
:::

:::danger Mismatched Callback Signatures
If your callback name doesn't match the string passed to `[CodeGenerator]`, or if the parameters don't exactly match the arguments of the wrapped method, the engine will fail to compile the generated code. Ensure your arguments (like `string message, int count`) exactly align with the method being wrapped.
:::

## Related Pages
- [Code Basics](index.md)