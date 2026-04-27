---
title: "Engine Definitions"
icon: "📖"
updated: 2026-04-25
sources:
  - engine/Definitions/readme.txt
  - engine/Definitions/engine.def
  - engine/Definitions/Definitions.csproj
  - engine/Tools/InteropGen/
created: 2026-04-27
---

# Engine Definitions

The `engine/Definitions/` directory holds the input files for **InteropGen** — the native↔managed binding generator. There is no C# code here; only `.def` files written in InteropGen's DSL plus a near-empty `Definitions.csproj` whose only job is to be referenceable by the build pipeline.

## What's in the folder

| Path | Purpose |
|---|---|
| `engine/Definitions/Definitions.csproj` | Essentially empty; no source compiled here. |
| `engine/Definitions/readme.txt` | Documents the `[Handles:T]` class directive. |
| `engine/Definitions/engine.def` | Top-level: binds `engine2.dll`. |
| `engine/Definitions/assetsystem/` | Bindings for the asset compiler. |
| `engine/Definitions/animgraph/` | AnimGraph native API. |
| `engine/Definitions/common/` | Shared types pulled in by other defs. |
| `engine/Definitions/engine/` | Native engine subsystems split by topic. |
| `engine/Definitions/hammer/` | Hammer editor native API. |
| `engine/Definitions/modeldoc/` | ModelDoc native API. |
| `engine/Definitions/resources/` | Resource-system native API. |

Each top-level `.def` declares an `ident`, a target `nativedll`, output paths for the generated C++/managed glue, and a tree of `include` directives that pull in the per-topic sub-files.

## `.def` directive reference

| Directive | Value | Effect |
|---|---|---|
| `ident "name"` | string | Logical name of this binding set; used in symbol prefixes the generator emits. |
| `nativedll <name>.dll` | DLL name | The C++ DLL whose exports are being bound. |
| `exceptions "Type"` | fully-qualified C# type | The managed type binding errors are routed through. |
| `namespace "X.Y"` | namespace string | Where generated managed wrappers are placed. |
| `cpp "path"` | path | Relative output path for the generated C++ side. |
| `hpp "path"` | path | Relative output path for the generated C++ header. |
| `cs "path"` | path | Relative output path for the generated C# wrapper. |
| `include "name"` | path or glob | Pulls another `.def` into this binding set. |
| `managed static class Foo.Bar { ... }` | block | Declares a managed-side wrapper class with the listed signatures. |

## The `[Handles:Type]` class directive

`readme.txt` documents one class-level directive that's worth flagging because it looks like a C# attribute but isn't:

```text
[Handles:Sandbox.PhysicsBody]
```

Read this as a *`.def` annotation*. It marks the managed class `Sandbox.PhysicsBody` as participating in the **handle indirection**: native code never holds a raw pointer to the managed object; instead it holds an `int` handle, and the managed side maintains a lookup table.

The flow:

1. The native object is constructed; it calls back into managed code to allocate a handle.
2. The handle (an int) is stored on the native side.
3. When native passes the object across the boundary, it passes the int.
4. Managed code resolves the int to the live C# object via the handle table.
5. The native destructor calls back into managed to release the handle, freeing the C# object for collection.

This decouples GC movement and lifetime from the C++ side, which doesn't have a stable pointer model for managed memory.

## InteropGen pipeline

`.def` files are consumed by `engine/Tools/InteropGen/` (the generator project) to produce:

- **C++ glue** at the `cpp`/`hpp` paths each `.def` declares.
- **Managed wrappers** at the `cs` path — these end up under `Sandbox.Engine`'s `Interop.*.cs` files and are what `Sandbox.Engine` actually compiles against.

The hand-written interop layer in `engine/Sandbox.Engine/Core/Interop/` consumes the generated wrappers — see [Sandbox.Engine](sandbox-engine/index.md) for how that fits together.

## When you'd modify a `.def`

- Exposing a new native API to managed code.
- Renaming the namespace, exception class, or output paths for an existing binding.
- Adding or removing classes from the handle system.
- Splitting a large `.def` into topic sub-files via `include`.

After editing, regenerate via `Bootstrap.bat` (which invokes InteropGen). Wrappers regenerate in place; commit the result alongside the `.def` change.

## Common contributor pitfalls

:::warning Cyclic dependencies
The `Definitions` project has no upward dependencies on purpose. Adding a reference *from* it to another `Sandbox.X` project will break the build. If you need a type at this layer, define it here or in `Sandbox.System`.
:::

:::danger `.def` is not C#
`[Handles:T]` and the rest of the `.def` syntax look like C# but aren't parsed by the C# compiler. Don't paste these directives into `.cs` files — they only have meaning inside `.def` files consumed by InteropGen.
:::

## Related Pages

- [Engine Concepts → Definitions](../engine-concepts/definitions.md) — the conceptual walkthrough.
- [Sandbox.Bind](sandbox-bind.md) — Razor UI data-binding (different system, often confused with native interop).
- [Sandbox.Engine](sandbox-engine/index.md) — where the generated C# wrappers live.
- [Foundational Systems Architecture](../systems/foundations/architecture.md) — the broader layering picture.
