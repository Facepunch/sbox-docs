---
title: "Code Basics"
icon: "👶"
created: 2025-06-15
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
---

# Code Basics

The pages in this section cover the day-to-day mechanics of writing C# in s&box. They're shorter and more practical than the broader [Code](../index.md) overview — drop into whichever you need.

## Component shape

Almost everything you write is a `Component` derived from `Sandbox.Component`. Minimum useful example:

```csharp
using Sandbox;

public sealed class MoveComponent : Component
{
    [Property] public float Speed { get; set; } = 100f;

    protected override void OnUpdate()
    {
        GameObject.WorldPosition += Vector3.Forward * Speed * Time.Delta;
    }
}
```

Three things to notice:

- **`using Sandbox;`** — almost every engine type lives in the `Sandbox` namespace. Without this, your file won't see `Component`, `Vector3`, `Time`, etc.
- **`[Property]`** — exposes the value to the Inspector so designers can tune it visually. Add `[Range(0, 500)]`, `[Title("Speed")]`, etc. for richer editor controls.
- **`Time.Delta`** — multiply continuous changes by it so they're framerate-independent. Without this, your object moves faster on faster machines.

## What's in this section

| Page | When you need it |
|---|---|
| [Cheat Sheet](cheat-sheet.md) | Quick API reference — common methods for logging, transforms, GameObjects, component lookup |
| [Math Types](math-types.md) | `Vector3`, `Rotation`, `Transform`, `Angles` — including the quaternion gotchas |
| [Hotloading](hotloading.md) | The practical guide: pitfalls, IL fast-hotload, diagnosing slow hotloads |
| [API Whitelist](api-whitelist.md) | Which standard .NET APIs are blocked and what to use instead |
| [Console Variables](console-variables.md) | Exposing tunables to the developer console |
| [Code Generation](code-generation.md) | What the source generators produce and when to look at the generated code |
| [Unit Tests](unit-tests.md) | Writing tests that interact with engine types |

## Things you'll trip on early

**`Sandbox` namespace not found in your IDE.** Right-click your `.sbproj` in the editor and pick "Generate Solution" — that creates the `.csproj` files your IDE needs. Then reload the solution in your IDE.

**Compile errors block hotload entirely.** The editor console shows red errors; nothing recompiles until they're fixed. The fastest debug loop is: keep the editor console visible, save → look at console → fix → save again.

**`System.IO` blocked.** The sandbox doesn't let you read arbitrary files from disk. Use the engine's `FileSystem` (mounted addons, user data) — see [API Whitelist](api-whitelist.md) for the full sandbox boundary.

**Default field values won't change after hotload.** If you change `public float Speed = 100f;` to `200f` and save, instances that already exist keep `100f`. Hotload preserves runtime state. Restart the scene to see new defaults, or use property getters (`public float Speed => 200f;`) which re-evaluate.

## Read next

- [Code (parent)](../index.md) — the bigger picture (lifecycle, hot path, gotchas)
- [Component Lifecycle](../../scene/components/component-lifecycle.md) — what `OnUpdate`, `OnStart`, etc. mean
- [Property Attributes](../../editor/property-attributes.md) — every editor attribute (`[Property]`, `[Range]`, `[Group]`, `[ShowIf]`, `[FilePath]`, etc.)
