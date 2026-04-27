---
title: "Sandbox.Generator"
icon: "✨"
sources:
  - engine/Sandbox.Generator/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Generator

The `Sandbox.Generator` namespace houses the internal Roslyn Source Generators that power s&box's boilerplate-free API, specifically handling property replication, RPCs (Remote Procedure Calls), and component serialization.

## Quick Start Example

As an engine developer, you might need to inspect the C# code output by the generator. The generator intercepts user code during compilation:

```csharp
// User writes this:
[Property, Sync] public float Health { get; set; }

// Sandbox.Generator conceptually outputs this hidden partial class:
partial class MyComponent
{
    protected override void __BuildNetworkTable()
    {
        // Registers 'Health' with the underlying network delta system
        NetworkTable.Register<float>( "Health", ref _health_backing_field );
    }
}
```

## Common Patterns

1. **Partial Classes:** The generator requires user classes to be `partial` (though `Sandbox.CodeUpgrader` can often automatically fix this if the user forgets). It generates a companion file containing all the wiring.
2. **RPC Dispatch Generation:** When a user marks a method `[Rpc.Broadcast]`, the generator creates a hidden method that serializes the arguments, sends them over the network, and another hidden method that deserializes and invokes the original function on the receiving end.

## Visual Diagnostics
If the source generator fails or produces invalid code, the resulting errors will appear in the Editor Console just like standard C# errors, but they will point to a virtual file name (e.g., `MyComponent.sbgen.cs`). Engine developers can dump these generated files to disk via debug compiler flags to inspect the exact output.

## Troubleshooting

:::warning
- **"Type must be partial to be generated":** The user forgot the `partial` keyword on a class containing `[Sync]` or `[Property]`.
- **"Unsupported Type for [Sync]":** The generator does not know how to serialize the type the user attempted to network. The user must provide a custom serializer or use a supported primitive.
:::

## Sample Validation
Look at `engine/Sandbox.Test.Generate/` where unit tests feed source code strings to the generator and assert that the output C# string perfectly matches the expected network boilerplate.

## Related Pages
- [Code Hotloading & Compilation](../scripting-code-workflows/code-hotloading.md)
