---
title: Engine Launcher
icon: "⌨️"
sources:
  - engine/Launcher/Launcher.cs
updated: 2026-04-25
created: 2026-04-27
---

# Engine Launcher

> **See also:** [Launcher Internals](../engine-internals/launcher.md) — engine-developer deep dive on the bootstrap flow. [Engine Launcher (Runtime Layout)](../engine-internals/runtime-layout/launcher.md) — placement of the launcher in the runtime startup sequence.

The Engine Launcher is the entry point executable that bootstraps the C# environment, resolves native dependencies, and starts the Source 2 engine loop.

## Quick Start Example

You do not interact with the launcher directly when making games. However, if you are building the engine from source, understanding its startup flow is critical. Here's the actual flow from `engine/Launcher/Launcher.cs`, condensed:

```csharp
using Sandbox;
using System;

public static class Program
{
    static string GamePath;        // folder containing sbox.exe
    static string ManagedDllPath;  // GamePath\bin\managed\
    static string NativeDllPath;   // GamePath\bin\win64\

    [STAThread]
    public static int Main()
    {
        // 1. Hook assembly resolution so managed DLLs load from bin/managed/
        AppDomain.CurrentDomain.AssemblyResolve += CurrentDomain_AssemblyResolve;

        // 2. Resolve paths from the executable location
        var exePath = Environment.ProcessPath;
        GamePath = System.IO.Path.GetDirectoryName( exePath );
        ManagedDllPath = $"{GamePath}\\bin\\managed\\";
        NativeDllPath = $"{GamePath}\\bin\\win64\\";

        // 3. Expose the engine path to MSBuild and unit tests
        Environment.SetEnvironmentVariable( "FACEPUNCH_ENGINE", GamePath,
            EnvironmentVariableTarget.User );

        // 4. Force native DLL lookups to hit our bin/win64/ first
        var path = Environment.GetEnvironmentVariable( "PATH" );
        Environment.SetEnvironmentVariable( "PATH", $"{NativeDllPath};{path}" );

        // 5. Defer to a separate method — Sandbox.Engine.dll cannot be touched
        //    until the paths above are in place.
        LaunchGame();
        return 0;
    }

    static int LaunchGame()
    {
        // Loads the C++ DLLs and binds the interop function table
        NetCore.InitializeInterop( GamePath );

        // Hand control to the Source 2 engine application loop
        var app = new SourceEngineApp( GamePath );
        app.RunLoop();
        return 0;
    }
}
```

## Common Patterns

1. **Path Resolution:** The launcher dynamically determines its execution path and sets up `ManagedDllPath` and `NativeDllPath`. It forces the OS to look in the game's `bin/win64/` directory for native DLLs first, preventing conflicts with other installed software.
2. **Assembly Resolution:** Because the engine DLLs are not in the exact same folder as the launcher executable, the launcher hooks into `AppDomain.CurrentDomain.AssemblyResolve` to manually find and load managed assemblies from the `bin/managed/` folder.
3. **Environment Variables:** The launcher sets the `FACEPUNCH_ENGINE` environment variable, which is utilized by MSBuild, unit tests, and other external tools to reliably locate the engine installation.

## Project Flavors

The `engine/Launcher/` directory contains several different project configurations that produce different executables:

*   **`Sbox`:** The standard client executable for players.
*   **`SboxDev`:** The development editor executable with the full toolset.
*   **`SboxServer`:** The headless dedicated server executable.
*   **`SboxStandalone`:** Used when exporting your game as a standalone product.
*   **`CrashReporter`:** A tiny utility that catches unhandled exceptions and uploads minidumps.

- **Missing Native DLLs:** If `engine2.dll` or other Source 2 native binaries are missing from `bin/win64/`, the launcher will crash with a `DllNotFoundException` when it attempts to call `NetCore.InitializeInterop`.

## Related Pages
- [Runtime Layout & Architecture](../engine-internals/runtime-layout/index.md)
