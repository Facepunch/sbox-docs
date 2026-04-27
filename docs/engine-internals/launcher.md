---
title: "Engine Launcher"
icon: "▶️"
sources:
  - engine/Launcher/Launcher.cs
updated: 2026-04-25
created: 2026-04-27
---

# Engine Launcher

> **See also:** [Launcher (Engine Concepts)](../engine-concepts/launcher.md) — high-level conceptual overview. [Engine Launcher (Runtime Layout)](runtime-layout/launcher.md) — startup-sequence placement and `.runtimeconfig.json` summary. Child pages [crash-reporter](launcher/crash-reporter.md), [sbox-launchers](launcher/sbox-launchers.md).

The `engine/Launcher` directory contains the executable entry points for the s&box engine. It is responsible for initializing the .NET runtime, setting up the host environment, resolving native bindings, and bootstrapping the core engine loop.

## Quick Working Example

```csharp
using Sandbox;
using Sandbox.Diagnostics;
using System;

// Fictionalized representation based on Launcher.cs
public static class Program
{
    [STAThread]
    public static int Main()
    {
        // 1. Resolve paths for the engine and DLLs
        var exePath = System.Environment.ProcessPath;
        var GamePath = System.IO.Path.GetDirectoryName( exePath );
        
        // 2. Load the C++ DLLs and initialize interop
        NetCore.InitializeInterop( GamePath );
        NativeEngine.EngineGlobal.Plat_SetModuleFilename( $"{GamePath}\\sbox.exe" );

        // 3. Start the Source Engine application loop
        var app = new SourceEngineApp( GamePath );
        app.RunLoop();

        return 0;
    }
}
```

## Common Patterns

1.  **Domain Event Resolution:** The launcher relies heavily on `AppDomain.CurrentDomain.AssemblyResolve` to hijack DLL loading so it can prioritize paths within the engine's managed binary folders before querying standard paths.
2.  **Delaying Core Loads:** Notice in `Launcher.cs` that the actual engine initialization (`LaunchGame()`) is separated into a distinct function. This prevents the CLR from eagerly trying to load `Sandbox.Engine.dll` at the start of `Main()` before the resolution paths are properly configured.
3.  **Native Path Prioritization:** The launcher explicitly prepends `NativeDllPath` to the system's `PATH` environment variable. This ensures any subsequent native module lookups by the OS load the engine's specific versions rather than accidentally finding conflicting generic DLLs on the machine.

## Performance

- **Startup Sequence Overhead:** The Launcher does synchronous file path probing and sets up .NET interop before kicking off the `SourceEngineApp`. While this takes a moment, it happens precisely once during process initialization and has zero impact on frame-to-frame performance.

- **Custom Resolution Logic:** The `CurrentDomain_AssemblyResolve` handler specifically strips out `.resources.dll` suffixes, treating them as standard `.dll`s for the purpose of path resolution, which is critical for correctly loading localized or resource-heavy assemblies at startup.

## Troubleshooting & Errors

:::warning Launch Failures
If the engine crashes immediately upon launch with no log output, the issue is almost always in the Launcher layer before the logging system has been fully initialized or because a native DLL failed to resolve. Check the Windows Event Viewer or run the executable via command line to catch `.NET Core Runtime` initialization failures.
:::

## Related Pages
* [.NET Core Hosting](../runtime-layout/launcher/netcore-hosting.md)
* [Engine Architecture](../systems/foundations/architecture.md)
