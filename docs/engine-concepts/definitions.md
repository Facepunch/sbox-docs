---
title: Engine Definitions
icon: "💻"
updated: 2026-04-25
sources:
  - engine/Definitions/readme.txt
  - engine/Definitions/engine.def
  - engine/Tools/InteropGen/
created: 2026-04-27
---

# Engine Definitions

The `engine/Definitions/` directory is **not C# source**. It's a set of `.def` files — InteropGen's domain-specific language — that describe how the native C++ Source 2 engine is exposed to managed C#. Each `.def` file declares native function signatures, namespace mappings, exception bindings, and class-handle relationships. `engine/Tools/InteropGen/` consumes these at build time and emits the C# wrapper code that `Sandbox.Engine` then ships.

## What a `.def` file looks like

From `engine/Definitions/engine.def`:

```text
ident "engine"
nativedll engine2.dll

// Class to receive interop binding exceptions
exceptions "Sandbox.Interop.BindingException"

// Generated wrappers go in this namespace
namespace "Managed.SandboxEngine"

cpp "../../src/engine2/interop.engine.cpp"
hpp "../../src/engine2/interop.engine.h"
cs  "../Sandbox.Engine/Interop.Engine.cs"

include "engine/*"
include "tier3"
include "common/*"

managed static class Sandbox.Engine.Bootstrap
{
    static void EnvironmentExit( int nCode );
}
```

Each directive tells InteropGen something specific:

| Directive | Effect |
|---|---|
| `ident` | Name of this binding set; appears in generated symbol prefixes. |
| `nativedll` | The C++ DLL whose exports are being bound. |
| `exceptions` | The fully-qualified C# type the generator routes binding exceptions through. |
| `namespace` | Namespace for the generated managed wrappers. |
| `cpp` / `hpp` / `cs` | Output paths (relative to the `.def`). The generator writes here. |
| `include` | Pulls another file's directives into this binding set. |
| `managed static class X { ... }` | Declares a managed-side class wrapping native callables. |

## The `[Handles:Type]` directive

The `Definitions/readme.txt` documents one class-attribute directive that's easy to mistake for a C# attribute:

```text
[Handles:Sandbox.PhysicsBody]
```

This is **`.def` syntax**, not C#. It marks a managed class as participating in the engine's handle system: native C++ holds an integer handle (not a raw pointer), and a lookup table on the managed side maps that handle back to the live C# object.

The lifecycle:

1. Native code constructs an `IPhysicsBody`. The native constructor calls back into managed code to allocate a handle (an `int`) and stores it on the native struct.
2. Whenever native passes the body across the boundary, it passes the integer.
3. Managed code looks the integer up in its handle table and resolves the real C# object.
4. When the native object's destructor runs, it calls back into managed to release the handle, allowing the C# object to be collected.

This indirection lets the C# garbage collector move and free objects freely while native code holds something stable.

## What's *not* in `engine/Definitions/`

- **No C# logic.** The `Definitions.csproj` is essentially empty — `<TargetFramework>net10.0</TargetFramework>` and that's it. No `.cs` files. The directory is purely the InteropGen input.
- **No public API surface for game devs.** The directives here matter only to whoever is wiring up native bindings. A game dev never reads or writes a `.def` file.

## When you'd touch this

You add or modify a `.def` file when:

- Exposing a new native C++ API to managed code.
- Adjusting the namespace, output path, or exception class for an existing binding.
- Adding a class to (or removing it from) the handle system.

After editing, run `engine/Tools/InteropGen/` (via `Bootstrap.bat` or the dedicated build target) to regenerate the C# wrappers. The new wrappers land in the `cs` path declared at the top of the `.def`.

## Related Pages

- [Engine Internals → Definitions](../engine-internals/definitions.md) — the per-project reference view.
- [Sandbox.Bind (Engine Concepts)](sandbox-bind.md) — the data-binding system, **not** the native interop layer (despite the similar name).
- [Engine Internals overview](../engine-internals/index.md) — where the generated C# wrappers live.
