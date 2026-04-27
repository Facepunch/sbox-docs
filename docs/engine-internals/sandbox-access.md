---
title: Sandbox.Access
icon: "🛡️"
sources:
  - engine/Sandbox.Access/AccessControl.cs
  - engine/Sandbox.Access/AssemblyAccess.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Access

The `Sandbox.Access` namespace handles the security and sandboxing of user-generated C# code running in the s&box engine. It is responsible for analyzing compiled assemblies to ensure they do not perform malicious actions or use forbidden APIs before they are executed.

## Quick Working Example

When a new addon is downloaded or compiled, the engine's core compilation pipeline passes the raw bytes of the assembly to the AccessControl system:

```csharp
using Sandbox;
using System.IO;

public class SecurityManager
{
    private AccessControl _accessControl = new AccessControl();

    public bool LoadAddonAssembly( byte[] assemblyBytes )
    {
        using var stream = new MemoryStream( assemblyBytes );
        
        // Check if the assembly uses any forbidden APIs
        var result = _accessControl.VerifyAssembly( stream, out var trustedStream );

        if ( result.Success )
        {
            // The assembly is safe. trustedStream can now be used to actually load the code
            Log.Info("Assembly verified and trusted!");
            return true;
        }
        else
        {
            foreach ( var error in result.Errors )
            {
                Log.Error( $"Security Violation: {error}" );
            }
            return false;
        }
    }
}
```

## Common Patterns

1.  **IL Instruction Walking:** Inside `AssemblyAccess.Touch.cs`, the system walks every single IL instruction in the compiled assembly to find method calls (`OpCodes.Call`, `OpCodes.Callvirt`). It then checks those targeted methods against the AccessRules dictionary.
2.  **Caching Results:** Analyzing IL is a CPU-intensive process. `AccessControl.cs` hashes the incoming assembly using `SHA256.Create()` and maintains a `SafeAssemblies` dictionary cache mapping assembly names to their hashes. If a known-good assembly is loaded a second time and matches the cached hash, it bypasses the IL walk.
3.  **Global Assembly Cache Hijack:** `AccessControl` implements the `Mono.Cecil.IAssemblyResolver` interface to act as a custom assembly resolver that overrides standard .NET behavior via `AccessControl.Resolve.cs`. It ensures that any dependencies the custom code asks for (e.g., `System.IO` or `Sandbox.Engine`) are loaded exclusively from the engine's safe directory, preventing DLL planting attacks. When attempting to resolve an assembly, it will first check the currently tracked dynamic assemblies (`Assemblies` dictionary), and if it falls back to a system library, it restricts access specifically to `Sandbox.*`, `System.*`, and `Microsoft.AspNetCore.Components` assemblies loaded via the `GlobalAssemblyCache`.

## Troubleshooting

:::warning Modifying Access Rules
If you are an engine developer adding new functionality to the C# API, you must explicitly whitelist your new types if you want community games to use them.
- If you forget to update the `AccessRules` in the `Rules/` directory, community developers will receive compilation errors stating the type is "Not Allowed." 
- **DO NOT** blanket-whitelist namespaces like `System.Reflection` or `System.Runtime.InteropServices`, as this entirely defeats the sandbox.
:::

## Related Pages
- [.NET Core Hosting](../runtime-layout/launcher/netcore-hosting.md)
- [Code Compiling](sandbox-compiling.md)
