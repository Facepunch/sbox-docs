---
title: "Sandbox.Access/Modifier"
icon: "🔨"
sources:
  - engine/Sandbox.Access/Modifier/AssemblyModifier.cs
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Access/Modifier

> **Status:** The single class here, `AssemblyModifier`, is currently unused. The source comment says: *"Not currently used anywhere right now, but we may need it in the future."* This page documents what's there for the case where someone needs to wire it back in.

`AssemblyModifier` is a Mono.Cecil-based utility for rewriting `.dll` bytes in memory. It's parked: built, kept, but not on any execution path. If you're tracing how the sandbox actually verifies user code, you want [Sandbox.Access/Rules](../sandbox-access-rules/index.md) — that's the live one.

## What it can do

```csharp
var mod = new AssemblyModifier
{
    Rename = "MyRenamed",                              // optional: rewrite assembly name
    ChangeReference = new() { ["OldRef"] = "NewRef" }  // optional: redirect references
};

byte[] modified = mod.Modify( originalBytes );
```

Two operations:

- **`Rename`** — replaces `AssemblyDefinition.Name` with a new `AssemblyNameDefinition`, preserving the original version.
- **`ChangeReference`** — walks `MainModule.AssemblyReferences` and renames any matching reference. Useful if you wanted to swap a default library reference for an engine-specific implementation.

The `Modify(byte[])` method round-trips through Mono.Cecil's `AssemblyDefinition.ReadAssembly` / `Write`, so the output is a freshly-serialised assembly. That's heavier than pure analysis — only use this if you actually need to mutate, not inspect.

## The `CustomResolver`

`AssemblyModifier` has a private `CustomResolver : DefaultAssemblyResolver` that resolves cross-assembly references against `AppDomain.CurrentDomain.GetAssemblies()`, falling back to no-result on failure. The intent is "only resolve things we already have loaded." If you reactivate this class, that resolver is what keeps reference rewrites from triggering unbounded disk lookups.

## Related

- [Sandbox.Access](../sandbox-access.md) — the active sandbox boundary.
- [Sandbox.Access/Rules](../sandbox-access-rules/index.md) — the verifier that's actually on the hot path.
