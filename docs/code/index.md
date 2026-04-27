---
title: "Code"
icon: "💼"
created: 2025-06-15
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
---

# Code

You write s&box games in C#. There's no separate scripting language, no visual-only path you can't escape from — the same C# you write for components is what runs in the engine. This section is about the things you'll bump into when you actually start writing it.

```csharp
using Sandbox;

public sealed class HelloComponent : Component
{
    protected override void OnUpdate()
    {
        GameObject.WorldPosition += Vector3.Forward * 50f * Time.Delta;
    }
}
```

That's a real, working component. Save the file, the engine recompiles it, and the next time you click Play your object moves.

## What's distinctive about s&box C#

A few things differ from "C# in any other context":

- **Hotload.** When you save a `.cs` file, the engine compiles a new assembly in memory and swaps your live state into it. You don't restart anything. Most code edits — including adding `[Property]` fields — apply within milliseconds. See [Hotloading](code-basics/hotloading.md).
- **Sandboxed APIs.** Not every .NET API is allowed. `System.IO`, raw sockets, native interop, `Reflection.Emit` — blocked. The engine provides safe equivalents (`FileSystem`, `Sandbox.Http`, etc.). See [API Whitelist](code-basics/api-whitelist.md).
- **Property-driven editor UI.** Decorate a property with `[Property]`, plus optional `[Range]`, `[Title]`, `[Group]`, `[ShowIf]`, etc., and the editor's Inspector renders an appropriate control automatically. See [Property Attributes](../editor/property-attributes.md).
- **Reflection-driven everything.** ActionGraph, the Inspector, save/load, snapshots — they all read your types through `TypeLibrary`. Your job is to write idiomatic C#; the engine surfaces it everywhere.

## Read in roughly this order

1. [Code Basics](code-basics/index.md) — the cheat sheet, the math types, what's actually in your project's `.cs` files, hotload day-to-day.
2. [API Whitelist](code-basics/api-whitelist.md) — which standard .NET APIs are blocked and what to use instead.
3. [Math Types](code-basics/math-types.md) — `Vector3`, `Rotation`, `Transform`, `Angles`. Gotchas around quaternions vs Euler.
4. [Hotloading (developer guide)](code-basics/hotloading.md) — the practical pitfalls you'll hit (changing default field values, dictionaries with stale `Equals`, delegates to lambdas, reflection caches).
5. [Console Variables](code-basics/console-variables.md) — exposing tunables to the developer console.
6. [Code Generation](code-basics/code-generation.md) — how `[Sync]`, `[Property]`, etc. produce the generated wrappers you'll see in build output.
7. [Unit Tests](code-basics/unit-tests.md) — how to test code that touches engine types.
8. [Libraries](libraries.md) — sharing code across projects.

For deeper architectural views (how hotload works internally, the reflection system) see [Code Hotloading & Compilation](../scripting-code-workflows/code-hotloading.md) and [Type Library & Reflection](../scripting-code-workflows/reflection.md).

## Things people get wrong early

**"My changes aren't hotloading."** Almost always one of two things: you didn't save the file, or the project has compilation errors that pause the hotloader. Check the editor console — red errors block hotload entirely.

**`NullReferenceException` from another component.** Either your `[Property]` reference wasn't assigned in the Inspector (it'll be null at runtime), or `Components.Get<T>()` returned null because the component you wanted lives on a different GameObject. Use `[RequireComponent]` on a property if you want the engine to ensure it's there.

**Destroyed object reference still has a value.** When a GameObject is destroyed, references you held don't auto-null — they become invalid. Check `obj.IsValid()` before using a reference that might've been destroyed (especially in async code that yielded).

**Editor-namespace code in gameplay scripts.** `Editor.*` types are stripped from runtime builds. Using them in a `Component` works in the editor, fails to compile when you export. Keep editor extensions in folders that compile only for editor.

## The hot path: `OnUpdate`

`OnUpdate` runs every rendered frame for every enabled component. At 144 FPS, allocations or heavy queries inside it compound fast. The unbendable rules:

- Don't `new` anything you can avoid (no `new List<T>()` per frame, no `string.Format` if you can help it).
- Don't `Scene.GetAllComponents<T>()` — cache once in `OnStart`.
- Don't load assets — they should be assigned via `[Property]` or loaded once.
- Multiply by `Time.Delta` for anything that should be framerate-independent.

If you need physics-step-aligned logic, override `OnFixedUpdate` instead.

## Related Pages

- [Scene System](../scene/index.md) — how `Component` fits into the GameObject tree
- [Architectural Overview](../architecture.md)
- [Sweeper Sample](../build-games/samples-templates/sweeper.md) — a real game's worth of code to read
