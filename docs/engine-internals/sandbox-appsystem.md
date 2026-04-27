---
title: "Sandbox.AppSystem"
icon: "⚙️"
sources:
  - engine/Sandbox.AppSystem/AppSystem.cs
  - engine/Sandbox.AppSystem/EditorAppSystem.cs
  - engine/Sandbox.AppSystem/StandaloneAppSystem.cs
  - engine/Sandbox.AppSystem/TestAppSystem.cs
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.AppSystem

When you launch `sbox-dev.exe`, the editor doesn't appear because some `Main()` somewhere said "show window." It appears because something concrete and traceable happens, and that something is `AppSystem.Run()`.

`AppSystem.Run()` is the actual main loop of every s&box process. Open `engine/Sandbox.AppSystem/AppSystem.cs:90` and you can read it in 35 lines:

```csharp
public void Run()
{
    SetupEnvironment();
    Application.TryLoadVersionInfo( Environment.CurrentDirectory );

    Init();   // GC settings, NetCore.InitializeInterop, native interop comes online

    NativeEngine.EngineGlobal.Plat_SetCurrentFrame( 0 );

    while ( RunFrame() )                            // ← this is the engine's heartbeat
    {
        BlockingLoopPumper.Run( () => RunFrame() );
    }

    Shutdown();
}
```

`RunFrame()` is `protected virtual` and ultimately calls `EngineLoop.RunFrame(_appSystem, out wantsToQuit)`. That's where every frame's input → physics → render → networking dispatch happens. Hot-loop everything in s&box descends from this call.

## What an AppSystem actually does

Three things, in order:

1. **System requirements check.** `TestSystemRequirements()` runs first and aborts with a `MessageBox` if the CPU doesn't support AVX, or if a core dependency is missing.
2. **Boot.** `Init()` puts the runtime into `SustainedLowLatency` GC mode and calls `NetCore.InitializeInterop`, which lights up the C++ ↔ C# bridge. Subclasses extend `Init()` to bring up additional subsystems (the editor, the menu, the game instance, etc.).
3. **Tick until exit.** `Run()` calls `RunFrame()` until it returns false. Then `Shutdown()` runs the teardown sequence (it's long — see below).

## Subclasses (one per launch mode)

The AppSystem variants live as siblings in `engine/Sandbox.AppSystem/`. Each one corresponds to a different launcher target:

| Class | When it runs | Adds to base behavior |
|---|---|---|
| `EditorAppSystem` | `sbox-dev.exe` | Boots the editor (Inspector, Mapping, Hammer integration), creates the Qt main window, brings up Sandbox.Tools. |
| `QtAppSystem` | Standalone Qt-based tools | Provides the Qt event-loop integration without bringing up the full editor. |
| `StandaloneAppSystem` | Exported standalone games | Strips editor functionality; sets `AppSystemFlags.IsStandaloneGame` and calls `Standalone.Init()`. |
| `TestAppSystem` | `dotnet test` runs | Lightweight: sets `AppSystemFlags.IsUnitTest`, doesn't bring up rendering, used by the unit-test harness. |
| `ToolAppSystem` | Build pipeline tools (`SboxBuild`, asset processors) | No game loop — used to host CoreCLR for one-shot utilities like the shader compiler. |

When you're hunting a "this only happens in the editor" bug, the place to start is `EditorAppSystem.cs`. When you're hunting a "this only happens in standalone," start at `StandaloneAppSystem.cs`. The base class never runs alone.

## The bootstrap sequence in `InitGame`

For game-running variants (Editor, Standalone, Test), `Run()` eventually calls `InitGame(createInfo)`, which is the more interesting path:

1. `CMaterialSystem2AppSystemDict.Create` — creates the native AppSystemDict on the C++ side.
2. Mode flags set: `SetInToolsMode()`, `SetInTestMode()`, `SetInStandaloneApp()`, `SetDedicatedServer(true)` — chosen by `AppSystemFlags`.
3. `SourceEnginePreInit` and `SourceEngineInit` — native engine initialization. Fails here are unrecoverable.
4. `Bootstrap.PreInit` and `Bootstrap.Init` — managed-side AppSystem boot in dependency order.

After this returns, the engine is fully up: scenes can be loaded, the menu can render, packages can be mounted, etc.

## The Shutdown sequence (why it's long)

`Shutdown()` in `AppSystem.cs` is ~180 lines. The reason: native resources held through C# references won't release just because the process is exiting. The Shutdown method has to manually tear down in the right order or the process will hang.

The sequence (condensed):

1. Fire shutdown analytics, save ConVars, signal `IToolsDll.Exiting()` / `IMenuDll.Exiting()` / `IGameInstanceDll.Exiting()`.
2. Drain queued scene/world deletes (`SceneWorld.FlushQueuedDeletes`).
3. Walk every loaded assembly that references `Sandbox.Engine` and **null out static references** to `Texture`, `Model`, `Material`, `ComputeShader`, `CommandList`, `UI.Panel` — without this, the GC can't collect the native handles.
4. Material/font/render-target/render-pipeline shutdown.
5. `GC.Collect()` + `GC.WaitForPendingFinalizers()` (twice — finalizers queue more work).
6. `SourceEngineShutdown` on the native side, then `Free()` each native interop module.

If you're working on AppSystem and you see "leaked scene" warnings on shutdown, that's step 5 not catching everything. The fix is usually finding which static somewhere holds a `SceneWorld`/`SceneModel` reference and making sure it gets nulled in step 3.

## Edge cases worth knowing

- **AVX requirement.** `TestSystemRequirements` exits the process if AVX isn't supported. This is checked *before* native interop comes up, so the check has to be self-contained — that's why it uses `MessageBox` directly via P/Invoke.
- **Steam DLL pinning.** `LoadSteamDll()` explicitly loads `bin/win64/steam_api64.dll` to prevent OS-level DLL search from finding a stale or pirate copy.
- **Crash path.** If anything throws inside `Run()`, control transfers to `CrashShutdown()` which is best-effort: save ConVars, flush error reporter, then `Plat_ExitProcess(1)`. No finalizers, no DLL destructors, no minidump generation here — those happen in the native crash reporter, not here.

## Common contributor pitfalls

:::warning Tick-order assumptions
The order in which subsystems initialise inside `Bootstrap.Init` matters. Adding a new subsystem that depends on, say, the rendering system, but registers before it, will null-deref on first frame. AppSystem dependency order is declared, not source-order — see `Sandbox.AppSystem/AppSystemFlags.cs` and the `Bootstrap` class.
:::

:::warning Blocking the main thread
Code at this level runs on the main engine thread. Any blocking operation here freezes the entire process. Use `Task` and marshal back via `MainThread.RunQueues()` if you need to do work that touches scene state.
:::

## Related

- [.NET Core Hosting](runtime-layout/launcher/netcore-hosting.md) — what runs *before* `AppSystem.Run`.
- [AppSystem Architecture](runtime-layout/appsystem.md) — the dependency-graph view.
- [Engine Launcher (Internals)](launcher.md) — the native side that calls into us.
