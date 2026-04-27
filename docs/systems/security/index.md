---
title: "Security & AccessControl"
icon: "🔒"
sources:
  - engine/Sandbox.Access/AssemblyAccess.cs
created: 2026-04-27
updated: 2026-04-27
---

# Security & AccessControl

s&box runs untrusted user code (addons and downloaded games) within a strict sandboxed environment using an IL-level **AccessControl** system to prevent malicious behavior like unauthorized file access, network requests, or memory corruption.

## How it Works

The verifier uses a **default-deny whitelist model**. Every time a package assembly (`package.*.dll`) is loaded or compiled, the engine walks the Intermediate Language (IL) instructions and extracts every type, method, and field being "touched." If the touched member isn't explicitly defined in a whitelist regex rule, the assembly is rejected and will not run.

```csharp
// Example of what IS allowed:
var pos = new Vector3(0, 0, 0); // System.Numerics.Vector3 is whitelisted
Log.Info("Hello World");        // Sandbox.Engine public API is whitelisted

// Example of what is NOT allowed:
var p = new System.Diagnostics.Process(); // Process is NOT whitelisted -> Assembly Rejected
System.IO.File.Delete("C:\\Windows\\System32\\cmd.exe"); // File is NOT whitelisted -> Assembly Rejected
```

## How It Works

1. **Upload / Compile**: When a developer compiles C# or a user downloads a package, an assembly (`package.*.dll`) is generated.
2. **IL Verification**: Before loading, `AccessControl.VerifyAssembly` parses the DLL using Mono.Cecil.
3. **Touching Members**: The walker examines every instruction. For example, if it sees `call System.IO.File::Exists`, it logs a "touch" for `System.IO.File.Exists(System.String)`.
4. **Matching Rules**: The extracted touch strings are evaluated against a predefined set of regular expressions (the whitelist).
5. **Accept / Reject**: If any instruction touches a member that doesn't match a rule, verification fails.

## Whitelist Mechanics

The whitelist entries rely on regular expressions.

*   `Sandbox.Engine`, `Sandbox.System`, and other core s&box APIs are heavily whitelisted by default to allow game logic.
*   Types with the `*` wildcard (e.g., `System.Object*`) match everything starting with that string. *(Note: This can sometimes be greedy and match unintended siblings.)*

## Troubleshooting

:::danger Assembly Rejected
If your code fails to compile or load with an "Assembly Rejected" error, it means you are trying to use a .NET class or method that is not whitelisted by the Sandbox. You must find an alternative provided by the `Sandbox.*` namespaces, or request the type be whitelisted if it is purely computational and safe.
:::

:::warning Avoid Unsafe Code
Do not attempt to use `unsafe` blocks, pointers, or `Marshal` operations in published packages. They are strictly blocked to protect the host environment.
:::

## Related Pages
- [Code Basics](../../code/index.md)
