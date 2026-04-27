---
title: "Editor Debug Tools"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Debug/TextureResidencyInfo.cs
created: 2026-04-27
updated: 2026-04-27
---

# Editor Debug Tools

The `Sandbox.Tools.Debug` subsystem provides editor-side utilities for inspecting the running state of the engine. It is primarily used to surface low-level native engine data (like GPU memory allocation) to the managed C# tools environment.

## Quick Start Example

As an engine contributor, you can query `TextureResidencyInfo` to retrieve a list of all textures currently loaded into GPU memory. This is particularly useful when building custom diagnostic windows for memory profiling.

```csharp
using Editor;

public class MyMemoryProfiler
{
    public void PrintResidentTextures()
    {
        // Query the native render device for all loaded textures
        var textures = TextureResidencyInfo.GetAll();

        foreach ( var tex in textures )
        {
            Log.Info( $"Texture: {tex.Name} | GPU Mem: {tex.Loaded.MemorySize / 1024} KB" );
        }
    }
}
```

## Troubleshooting

:::warning
- **Memory Leaks during Native Interop:** If you modify `GetAll()` to extract new native objects but fail to call `DeleteThis()` on the underlying `CUtlVector` wrappers, the editor will leak unmanaged memory. Always ensure native vectors are explicitly destroyed after the managed translation is complete.
:::

## Related Pages
- [Editor Application Shell](editor.md)
