---
title: "Sandbox.Filesystem"
icon: "📁"
sources:
  - engine/Sandbox.Filesystem/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Filesystem

The `Sandbox.Filesystem` namespace provides the managed implementation of the virtual filesystem.

## Quick Working Example

When an engine developer needs to expose a new internal directory to the virtual filesystem securely, they interact with this subsystem.

```csharp
using Sandbox.Filesystem;

internal class InternalFileSystemConfig
{
    public void MountInternalData()
    {
        // Conceptual: Engine code mounting a native OS directory securely
        // into the virtual file system.
        var dataFs = new NativeFileSystem( "C:/sbox/internal_data" );
        
        // Ensure no absolute paths escape this root
        dataFs.SanitizePaths = true; 
        
        // Add to the global filesystem registry
        FileSystem.Add( "internal", dataFs );
    }
}
```

## Troubleshooting

:::warning
- **"UnauthorizedAccessException":** If a game developer tries to write to `FileSystem.Mounted` (which is read-only) or attempts to break out of the sandbox using `System.IO`, the request is blocked here.
- **"File not found" across addons:** If an asset exists in Addon A, but Addon B cannot read it, the engine developer must verify the mounting order and ensuring both addons are correctly registered in the unified Virtual File System map.
:::

## Related Pages
- [Virtual Filesystem Architecture](../systems/file-system/architecture.md)
