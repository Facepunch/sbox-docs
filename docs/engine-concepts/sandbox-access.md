---
title: Access Control & Sandboxing
icon: "🛡️"
sources:
  - engine/Sandbox.Access
updated: 2026-04-25
created: 2026-04-27
---

# Access Control & Sandboxing

> **See also:** [Sandbox.Access (Internals)](../engine-internals/sandbox-access.md) — engine-developer deep dive with `AccessControl.cs` / `AssemblyAccess.cs` walkthrough.

The `Sandbox.Access` system is the engine's built-in security layer. It statically analyzes compiled C# assemblies (DLLs) before they are allowed to execute, ensuring that user-generated content (Addons) cannot perform malicious actions on a player's machine.

## Quick Start Example

The Access Control system is entirely internal to the engine. You do not interact with it directly when making a game. Instead, you interact with its *rules* when your code gets blocked from compiling.

```csharp
using Sandbox;
using System.IO;

public sealed class BadComponent : Component
{
    protected override void OnStart()
    {
        // THIS WILL FAIL TO COMPILE OR LOAD IN AN ADDON
        // The Access Control system prevents addons from using System.IO.File directly
        // to prevent malware from reading/writing arbitrary files on the player's C: drive.
        
        // File.ReadAllText( "C:/passwords.txt" ); 
        
        // Instead, you must use the engine's safe virtual filesystem:
        var text = FileSystem.Data.ReadAllText( "my_save.txt" );
    }
}
```

## Common Patterns

1. **Assembly Verification:** The `VerifyAssembly` method reads the byte stream of a DLL. If it passes all safety checks, it returns a `TrustedBinaryStream`. Only trusted streams are allowed to be loaded into the active .NET `AppDomain`.
2. **Whitelist Configuration:** The engine maintains an internal list of "safe" assemblies and namespaces. Standard collections (`System.Collections.Generic`), math (`System.Math`), and basic types are allowed.
3. **Hotload Security:** Even during fast hotloading, the Access Control system runs on the newly compiled delta-assemblies to ensure no malicious code was sneaked in during a live edit.

## Allowed vs Disallowed

**Generally Allowed:**
*   Most of `System` (Math, String, primitives)
*   `System.Collections`, `System.Linq`
*   `System.Text.Json`
*   All `Sandbox.*` engine APIs

**Strictly Blocked:**
*   `System.IO` (Direct file access — use `Sandbox.FileSystem`)
*   `System.Reflection` (Can be used to bypass private variable security)
*   `System.Net` (Direct sockets — use `Sandbox.Http` or `Sandbox.WebSocket`)
*   `System.Threading.Thread` (Direct thread management — use engine async/Tasks)
*   `System.Runtime.InteropServices` (P/Invoke to native DLLs)

## Troubleshooting

:::warning Access Denied Errors
- **"Type 'System.IO.File' is not allowed":** You are using a blocked .NET feature. You must find the s&box equivalent, usually located in `Sandbox.FileSystem`, `Sandbox.Http`, etc.
- **Using 3rd Party Nuget Packages:** If you import a Nuget package into your addon (like a JSON parser or database client), it will likely be blocked by Access Control because those packages often rely on Reflection or File IO under the hood. You generally cannot use complex 3rd party .NET libraries in s&box addons.
:::

## Related Pages
- [File System](../systems/file-system/index.md)
- [Runtime Layout](../engine-internals/runtime-layout/index.md)
