---
title: Hotload & Compile
icon: "🔄"
created: 2026-04-25
updated: 2026-04-25
---

# Hotload & Compile

Code changes not appearing, hotload errors, "Type can't be found", red console errors. The first place to look when you've edited C# and nothing changed.

## Code changes not picking up

### "I edited my code but nothing changed in-game"

Hotloading pauses entirely if there are *any* compilation errors anywhere in the project.

- **Check:** Editor console for red error messages. Fix every red line first.
- **Check:** the file is actually saved to disk. Hotload watches disk, not the IDE buffer.
- **Check:** you're editing the file in the active project, not a sibling addon you aren't depending on.

**Deep dive:** [Hotloading](../code/code-basics/hotloading.md).

### "Default values for my [Property] aren't updating"

Inspector values are serialized when the component was added. The default value in code is only used the first time.

- **Fix:** delete and re-add the component to pick up the new default.
- **Fix:** clear the override in the Inspector (right-click the field → Reset to default).

### "Type 'X' could not be found"

Either a missing `using`, a missing dependency, or the class genuinely doesn't compile.

- **Check:** `using Sandbox.UI;` etc. at the top of your file.
- **Check:** the addon defining the type (e.g., `facepunch.base`) is listed in your `.sbproj` dependencies.
- **Check:** that addon's own code has no compile errors — if it failed to compile, its types disappear from your project too.

### "My component isn't showing up in Add Component"

Compile error somewhere — even an unrelated one — pauses the type registry.

- **Fix:** clear all red errors in the console.
- **Fix:** the class must be `public` and inherit from `Component`.
- **Check:** the class isn't in an Editor-only project if you expect it in the runtime.

**See also:** [Scene & Components](./scene-and-components.md).

## Code editor and IDE

### "Visual Studio / VS Code says it can't find the Sandbox namespace"

The `.sln` is stale or wasn't generated.

- **Fix:** in the s&box editor, right-click your `.sbproj` → "Generate Solution".
- **Fix:** ensure the s&box install path hasn't moved since you generated.

**Deep dive:** [Code Basics](../code/code-basics/index.md).

### "I launched s&box but there's no code editor"

s&box doesn't ship one. You install your own (VS, Rider, VS Code).

- **Fix:** install Visual Studio or VS Code, then in Editor settings → Code Editor pick the one you installed.

## Restricted APIs

### "Type 'System.IO.File' is not allowed"

You're using a blocked .NET API. The sandbox bans direct disk and network access.

- **Fix:** use `Sandbox.FileSystem` instead of `System.IO.File`.
- **Fix:** use `Sandbox.Http` instead of `HttpClient`.
- **Note:** Editor projects are not sandboxed — if the code only runs in the editor, putting it in an Editor project lifts the restriction.

**Deep dive:** [Security & Access Control](../systems/security/index.md), [Sandbox.Access](../engine-concepts/sandbox-access.md).

### "Assembly Rejected"

Same family — a referenced 3rd-party DLL uses banned APIs and got refused.

- **Fix:** find a sandboxed-friendly alternative, or move the code to an Editor-only project if it's a tooling library.

### "unsafe / pointer / Marshal blocked"

`unsafe` blocks, raw pointers, and `Marshal` are blocked in shipped packages.

- **Fix:** use safe equivalents (`Span<T>`, `MemoryMarshal` is also blocked, use Sandbox-provided types).

## Compile errors with code generators

### "Type must be partial to be generated"

You added `[Sync]` or `[Property]` to a class that isn't `partial`. The source generator can't extend it.

- **Fix:** mark the class `partial`.

### "Unsupported Type for [Sync]"

The generator doesn't know how to serialize the type you tried to sync.

- **Fix:** sync supported types only (primitives, common engine types, lists of those). Move complex objects to `[Property]` or use custom snapshots.

**Deep dive:** [Sandbox.Generator](../engine-internals/sandbox-generator.md), [Sync Properties](../systems/networking-multiplayer/sync-properties.md).

### "Mismatched Callback Signatures (CodeGenerator)"

Your callback name doesn't match the string in `[CodeGenerator]`, or argument types differ.

- **Fix:** match the callback signature exactly. Test on a simple wrapper first.

**Deep dive:** [Code Generation](../code/code-basics/code-generation.md).

### "Forgot to call Resume / Setter"

Wrapping a method but not calling `m.Resume()` (or for a property setter, `p.Setter( p.Value )`) means the original code is dead.

- **Fix:** always call `Resume()` or `Setter(...)` unless you intend to fully replace behavior.

## Hotload state loss

### "State on my static cache is gone after hotload"

Hotload swaps types. Static fields holding instances of user types get cleared.

- **Fix:** clear and rebuild caches in an `[Event.Hotload]` handler. The engine emits this event so you can rehydrate.
- **Engine internal:** `ReflectionCache<TKey, TValue>` exists for this purpose.

**Deep dive:** [Hotloading](../code/code-basics/hotloading.md).

### "Engine crash on hotload"

A static reference to a soon-to-be-replaced type prevents GC, but the engine tries to swap the type anyway.

- **Fix:** use weak references, or null out static caches in `[Event.Hotload]`.
- **Engine-side:** "Assembly Load Context cannot unload" means the leak is in engine code; usually a managed thread holding a user type.

**Deep dive:** [Sandbox.Hotload](../engine-internals/sandbox-hotload.md).

## Build / engine errors

### "Failed to emit assembly"

Roslyn pipeline failure — usually a custom source generator crashed.

- **Check:** you haven't installed an experimental Roslyn analyzer or generator in your project that's incompatible.

**Deep dive:** [Sandbox.Compiling](../engine-internals/sandbox-compiling.md).

### "Reference to type 'X' claims it is defined in 'Y' but it could not be found"

Mismatched assembly versions. Often: you upgraded a dependency but didn't recompile dependents.

- **Fix:** clean and rebuild from the top of the dependency chain.

### "DllNotFoundException / engine2.dll not found"

The launcher can't find native binaries. Usually only when running engine from source.

- **Fix:** verify s&box install integrity in Steam, or run `Bootstrap.bat` if you're working on the engine directly.

**Deep dive:** [Sbox Launchers](../engine-internals/launcher/sbox-launchers.md).

### "Failed to load hostfxr"

.NET runtime not installed or `Sandbox.NetCore` couldn't locate it.

- **Fix:** install the .NET SDK version listed in [System Requirements](../getting-started/system-requirements.md).

## Related Pages

- [Scene & Components](./scene-and-components.md)
- [Editor](./editor.md)
- [Runtime & Build](./runtime-and-build.md)
