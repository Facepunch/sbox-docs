---
title: "Sbox Launchers"
icon: "🚀"
sources:
  - engine/Launcher/Sbox/Launcher.cs
  - engine/Launcher/SboxBench/BenchmarkLauncher.cs
  - engine/Launcher/SboxDev/Launcher.cs
  - engine/Launcher/SboxServer/Launcher.cs
  - engine/Launcher/SboxStandalone/Launcher.cs
  - engine/Launcher/SboxProfiler/Program.cs
  - engine/Launcher/Shared/Startup.cs
updated: 2026-04-25
created: 2026-04-27
---

# Sbox Launchers

The Sbox Launchers act as the thin entry points that bootstrap the managed .NET Core host environment and delegate control to the core engine systems.

## Quick Working Code Example

Below is a simplified conceptual look at how a specific launcher implementation initializes the engine. Notice how the primary difference between launchers is often just the target name and specialized bootstrapping logic.

```csharp
using System;
using Sandbox;

namespace Sandbox.Launcher
{
    public static class Program
    {
        [STAThread]
        public static int Main()
        {
            // The shared launcher handles path resolution and initial setup
            var launcher = new Shared.Launcher( "sbox" );

            // Hooks to resolve assemblies correctly from the bin directory
            launcher.SetupAssemblyResolution();

            // Hands over control to Native engine / Sandbox.Engine
            return launcher.Run();
        }
    }
}
```

## Troubleshooting

:::warning Boot Failures
- **"Failed to load engine2.dll" / DLLNotFoundException:** This occurs if the launcher cannot find the native Source 2 binaries. The `Shared` launcher makes assumptions about directory structures (e.g., `../game/bin/win64/`). If you run the launcher outside of its expected folder, it will crash.
- **Assembly Resolution Failures:** If you add a new third-party dependency to `Sandbox.Engine` but fail to ensure the launcher can find it via the `AssemblyResolve` handler, the engine will crash silently or throw a `TypeLoadException` early in the boot process.
:::

## Firefox Export (SboxProfiler)
The `SboxProfiler` contains specific logic to export internal engine traces to the Firefox Profiler format. This allows engine developers to view detailed frame timings, lock contention, and memory allocations using the familiar `profiler.firefox.com` web interface. The `FirefoxExport/Markers/` directory defines how specific s&box events are translated into Firefox trace markers.

## Related Pages
- [Engine Launcher Overview](../launcher.md)
- [Crash Reporter](crash-reporter.md)
- [.NET Core Hosting](../../runtime-layout/launcher/netcore-hosting.md)
