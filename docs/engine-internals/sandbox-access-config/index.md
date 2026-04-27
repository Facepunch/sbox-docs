---
title: "Sandbox.Access/Config"
icon: "⚙️"
sources:
  - engine/Sandbox.Access/Config/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Access/Config

The `Config` module within `Sandbox.Access` manages the configuration schemas and whitelist generation used by the engine's security layer to sandbox user-generated code.

## Common Patterns

1. **Regex Compilation:** `AccessRules` takes human-readable wildcard definitions (like `System.Threading.Tasks.Task.*`) and compiles them into `Regex` filters (`RegexOptions.Compiled`) during initialization to optimize verification loops.
2. **Assembly Whitelisting:** `InitAssemblyList` builds a strict, hard-coded list of safe assemblies (e.g. `System.Private.CoreLib`, `Sandbox.Engine`, `Microsoft.AspNetCore.Components`), restricting what assemblies user code can reference.
3. **Blacklist vs Whitelist Integration:** A method or type reference is evaluated first against the `Blacklist`. If matched, access is denied. If it passes the blacklist, it must explicitly match a `Whitelist` rule to be approved. Rules prefixed with `!` are automatically parsed into the blacklist.

## Related Pages
- [Sandbox.Access](../sandbox-access.md)
