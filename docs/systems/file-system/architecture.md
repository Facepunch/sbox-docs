---
title: Virtual Filesystem Architecture
icon: "📁"
sources:
  - engine/Sandbox.Filesystem
created: 2026-04-27
updated: 2026-04-27
---

# Virtual Filesystem Architecture

The Virtual Filesystem (VFS) provides a secure, abstract layer over standard operating system disk I/O. It guarantees that game addons can interact with files seamlessly without risking unauthorized access to the user's host machine.

## Quick Start Example

You interact with the VFS using the static `FileSystem` API. Different storage contexts (like `Data` or `Mounted`) isolate file access.

```csharp
using Sandbox;

public sealed class DataSaver : Component
{
    protected override void OnStart()
    {
        // Write to the secure 'Data' directory
        FileSystem.Data.WriteAllText( "save.txt", "Player reached level 5" );
        
        // Read back from the same context
        Log.Info( FileSystem.Data.ReadAllText( "save.txt" ) );
    }
}
```

## Common FileSystem Contexts

- **`FileSystem.Data`:** The primary read/write storage area for the current game project. Use this for save files, player settings, and cached downloaded content.
- **`FileSystem.Mounted`:** An aggregate, read-only filesystem combining the currently active game project, all its mounted addons, and base engine files. Use this for loading assets (like models, sounds, and UI images).
- **`FileSystem.Organization`:** A shared read/write space accessible by all games published under your specific Organization Ident. Useful for sharing global player preferences across multiple titles you own.

## Integration with Sandbox.Mounting

The most powerful feature of the VFS is the **Aggregate FileSystem** (`FileSystem.Mounted`), which tightly integrates with the `Sandbox.Mounting` subsystem. 

When a user joins a server or mounts a legacy game (like Half-Life 2), the `MountHost` registers the archive with the VFS. Instead of dealing with disparate `.vpk` files or network streams, the VFS creates a seamless unified directory structure in memory.

If you request `FileSystem.Mounted.ReadAllBytes("materials/wall.vmt")`:
1. The VFS checks the local game project.
2. If not found, it queries registered addons in order of priority.
3. If an interceptor (like `Sandbox.Mounting.GoldSrc`) claims the path, the VFS delegates the read operation to the interceptor, which might parse the file from a compressed binary archive on-the-fly.

## Troubleshooting

:::warning
- **"System.IO.DirectoryNotFoundException / UnauthorizedAccess":**
  If you attempt to use standard .NET `System.IO` classes to access an absolute path like `C:/Windows/`, the engine's code analyzer or runtime sandbox will block the call. Always use `Sandbox.FileSystem`.
- **"File Exists but Read Fails":**
  If `FileSystem.Mounted.FileExists()` returns true but `OpenRead()` throws an error, it is likely the file is locked by another process (like a file browser or external editor). Ensure you properly `Dispose()` streams after reading them using `using` statements.
:::

## Related Pages
- [Mounting Internals](../../engine-internals/mounting.md)
