---
title: "Sandbox.NetCore"
icon: "🌐"
sources:
  - engine/Sandbox.NetCore/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.NetCore

The `Sandbox.NetCore` namespace manages the embedding and initialization of the .NET Core runtime within the native C++ engine.

## Quick Working Example
As an engine developer, if you need to pass additional configuration properties to the embedded .NET runtime during boot (like enabling a specific garbage collector mode), you would modify the bootstrap logic here.

```csharp
// Conceptual example of runtime initialization in Sandbox.NetCore
internal static class RuntimeBootstrapper
{
    public static void InitializeDotNet( string basePath )
    {
        var config = new HostConfiguration
        {
            AppPath = basePath,
            // Pass properties to the runtime here
            Properties = { { "System.GC.Server", "true" } } 
        };
        
        NativeHost.Start( config );
        Log.Info( "Managed runtime booted successfully." );
    }
}
```

## Troubleshooting

:::warning
- **"Failed to load hostfxr":** If you see this error when running the engine from source, it means the required .NET SDK/Runtime is missing from your system, or `Sandbox.NetCore` was unable to resolve the path to your dotnet installation. Ensure the correct .NET version is installed as per the prerequisites.
:::

## Related Pages
- [.NET Core Hosting](../runtime-layout/launcher/netcore-hosting.md)
