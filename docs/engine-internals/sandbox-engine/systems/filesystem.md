---
title: "Sandbox.Engine: Filesystem"
icon: "📁"
sources:
  - engine/Sandbox.Engine/Systems/Filesystem/FileSystem.cs
  - engine/Sandbox.Engine/Systems/Filesystem/EngineFileSystem.cs
created: 2026-04-27
updated: 2026-04-27
---

# Filesystem

The `Filesystem` subsystem provides a managed abstraction over native file I/O and Source 2's virtual mounting system. It handles loading data from disk, interacting with memory-only temporary files, and unifying paths across different game addons and the core engine.

## Core Concepts

The architecture operates heavily on `BaseFileSystem` instances, which represent a scoped tree of files (e.g., an addon's root, or a memory buffer).

### `Sandbox.FileSystem`
The public entrypoint primarily used by game developers. It offers static accessors to common, restricted filesystems:
*   **`Mounted`**: The aggregated view of all currently mounted addons, games, and core engine files.
*   **`Data`**: The persistent storage area specific to the current game/project.
*   **`OrganizationData`**: Shared persistent storage for all games under a specific organization.

### `EngineFileSystem`
This internal, engine-only static class manages the initialization of the raw directories before the game sandbox is fully established. It initializes sub-systems like `Config`, `Addons`, and the `EditorTemporary` space (`.source2/temp`). 

## Example: Engine Initialization

When the engine launcher first starts, it sets up the root paths based on whether it is running as a standalone game or as the development tools (`sbox-dev.exe`).

```csharp
using Sandbox;

// Internal engine bootstrapping
internal static void InitCoreStorage( string engineRoot )
{
    // Initializes the root LocalFileSystem and Mount aggregates
    EngineFileSystem.Initialize( engineRoot, skipBaseFolderInit: false );

    // Creates the /config path on disk and assigns it to EngineFileSystem.Config
    EngineFileSystem.InitializeConfigFolder();

    // Now EngineFileSystem.Root is available to map raw paths
    var rawText = EngineFileSystem.Root.ReadAllText( "gameinfo.txt" );
}
```

## Example: Creating a Memory Filesystem

Engine components and tools frequently need temporary files that shouldn't touch the disk (to avoid I/O blocking or clutter).

```csharp
using Sandbox;

internal void BuildTemporaryArchive()
{
    // Create an isolated filesystem in memory
    var memFs = FileSystem.CreateMemoryFileSystem();

    // Write to it exactly like a real disk
    memFs.WriteAllText( "compile_log.txt", "Build succeeded." );

    // Read back or pass the BaseFileSystem object to other engine APIs
    if ( memFs.FileExists( "compile_log.txt" ) )
    {
        Log.Info( memFs.ReadAllText( "compile_log.txt" ) );
    }
}
```

## Internal File Systems

The engine implements multiple variants of `BaseFileSystem` to handle specific I/O scenarios:

*   **`LocalFileSystem`**: Maps directly to physical paths on the OS disk (e.g., `C:\sbox\`).
*   **`MemoryFileSystem`**: Stores all written bytes in a managed RAM dictionary.
*   **`AggregateFileSystem`**: Used for `FileSystem.Mounted`. It doesn't hold files itself; instead, it contains a list of other `BaseFileSystem`s and queries them in priority order. This is how a game addon can "override" a file existing in `core/`.
*   **`PackageFileSystem`**: Interfaces directly with `.vpk` files or similar packaged archives, treating the archive as a root directory.
*   **`RedirectFileSystem`**: A wrapper that intercepts path requests and remaps them to a different physical path or alias before passing the call to an underlying file system.

## Troubleshooting

:::warning Root Path Assumptions
When working in `EngineFileSystem.Root`, be aware that the paths map literally to the installation directory. In standalone exports, `/addons/base/` does not exist in the same way it does in the development editor. Always check `Application.IsStandalone` if making assumptions about engine folder structures.
:::