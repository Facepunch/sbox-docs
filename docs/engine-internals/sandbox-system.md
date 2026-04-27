---
title: "Sandbox.System"
icon: "⚙️"
sources:
  - engine/Sandbox.System/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.System

The `Sandbox.System` namespace provides the foundational, engine-agnostic C# utility classes, math structs, collections, and extensions that power the rest of the s&box architecture.

## Quick Start Example

As a game developer or engine contributor, you use this namespace constantly without realizing it. Every vector math operation or log output relies on `Sandbox.System`:

```csharp
using Sandbox;

public void SystemExamples()
{
    // Using Math types defined in Sandbox.System
    Vector3 position = new Vector3( 100, 200, 0 );
    Rotation rotation = Rotation.FromAxis( Vector3.Up, 90 );
    Transform tx = new Transform( position, rotation );

    // Using Logging infrastructure
    Log.Info( $"Current Transform: {tx}" );

    // Using Byte serialization utilities for network messaging
    var stream = ByteStream.Create( 64 );
    stream.Write( position );
}
```

## Common Patterns

1. **Extension Methods:** `Sandbox.System` contains dozens of extensions for standard .NET types. For instance, extending `string` with methods like `.ToColor()` or `.ToFloat()` for rapid parsing.
2. **Ref Structs:** For extreme performance on hot paths, this namespace heavily utilizes `ref struct` and `ReadOnlySpan<T>` to process data (like network packets or geometry buffers) without triggering garbage collection allocations.

## Visual Diagnostics

The effects of `Sandbox.System` math operations are primarily visualized via the Editor's viewport Gizmos (which rely on `BBox` and `Transform` intersections) or by dumping `Log.Info` directly to the Developer Console.

## Troubleshooting

:::warning
- **"Cannot marshal type Vector3":** If an engine developer incorrectly modifies the `[StructLayout]` attributes within `Sandbox.System`, the .NET interop layer will crash when passing data to Source 2 C++.
- **Euler angle flipping:** A common game developer bug. Modifying `Angles` directly and applying them back to `Rotation` continuously can result in numerical drift or axis flipping. Always multiply `Rotation`s directly when possible.
:::

## Sample Validation

The `engine/Sandbox.Test.Unit/` project contains hundreds of math assertions validating that `Sandbox.System` operations perfectly mirror the expected outcomes of the native Source 2 C++ math libraries, preventing visual desyncs.

## Related Pages
- [Foundational Systems Architecture](../systems/foundations/architecture.md)
