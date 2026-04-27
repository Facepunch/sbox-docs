---
title: .NET Core Hosting
icon: "💻"
sources:
  - engine/Sandbox.NetCore/NetCore.cs
  - engine/Sandbox.Engine/Core/Interop/NetCore.cs
updated: 2026-04-25
created: 2026-04-27
---

# .NET Core Hosting

The C# half of the boot sequence. The native engine process loads `Sandbox.NetCore.dll` first, calls a single `[UnmanagedCallersOnly]` entrypoint, and from there the rest of the managed runtime comes online.

## What's actually happening

`engine/Sandbox.NetCore/NetCore.cs` is intentionally tiny — it has *no references* to `Sandbox.Engine.dll` or anything else managed. The reason is documented in the source comment:

> This dll is only called in instances where we want to initialize .net from c++. We purposefully don't reference any dlls here because it's loaded in its own isolated context, so if we load sandbox.engine, it technically won't be loaded in the main context and stuff will be weird.

So the shim's only job is:

```csharp
[UnmanagedCallersOnly]
public static int InitializeEngine( IntPtr gameFolderPtr )
{
    var gameFolder = Marshal.PtrToStringUTF8( gameFolderPtr );
    var managedFolder = $"{gameFolder}\\bin\\managed\\";

    // Reflectively load Sandbox.Engine.dll into the *main* context (not ours)
    var assembly = Assembly.LoadFrom( $"{managedFolder}Sandbox.Engine.dll" );
    var type = assembly.GetTypes().Where( x => x.Name == "NetCore" ).FirstOrDefault();
    var method = type.GetMethod( "InitializeInterop", BindingFlags.Static | BindingFlags.NonPublic );

    method.Invoke( null, new object[] { gameFolder } );
    return 0;
}
```

The `Sandbox.Engine.NetCore.InitializeInterop` it dispatches to is the second-stage entrypoint — that's where the real interop table population, AppSystem boot, and engine startup live.

## Boot order

1. **Native (`sbox.exe`)** starts up, loads CoreCLR.
2. **`Sandbox.NetCore.dll`** is loaded into a throwaway `AssemblyLoadContext`. Its `NetCore.InitializeEngine` is invoked from C++.
3. The shim reflectively loads `Sandbox.Engine.dll` into the **default** context, then calls into its `NetCore.InitializeInterop` (in `engine/Sandbox.Engine/Core/Interop/NetCore.cs`).
4. From that point everything is in the main context: AppSystem boots, scene system comes up, the launcher hands control to the engine loop.

The two-step dance exists so that the runtime context that holds the bulk of the engine isn't tainted by anything `Sandbox.NetCore` happened to reference.

## Why this is in its own DLL

The native side only knows the `Sandbox.NetCore.NetCore.InitializeEngine` symbol — it doesn't (and shouldn't) know about `Sandbox.Engine`'s assembly identity, its dependencies, or its load context. By keeping the shim free of references, native can call it safely without dragging the entire `Sandbox.Engine` graph into the wrong context.

If you find yourself wanting to add code to `Sandbox.NetCore`, you probably want `Sandbox.Engine.NetCore` (the second-stage init in `engine/Sandbox.Engine/Core/Interop/`) instead.

## Related

- [Engine Launcher (Internals)](../../launcher.md) — what the native side does before this point.
- [Sandbox.AppSystem](../../sandbox-appsystem.md) — what `InitializeInterop` brings up next.
