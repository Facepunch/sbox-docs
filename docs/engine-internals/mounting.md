---
title: "Mounting Internals"
icon: "🗜️"
sources:
  - engine/Mounting/
updated: 2026-04-25
created: 2026-04-27
---

# Mounting Internals

> **See also:** [Mounting (Engine Concepts)](../engine-concepts/mounting.md) — high-level conceptual overview. [Mounting (Runtime Layout)](runtime-layout/mounting.md) — placement of Mounting in the runtime startup sequence. [External Game Mounting (Systems)](../systems/mounting/index.md) — game-developer perspective on mounting GoldSrc/NS2/Quake content.

The Mounting subsystem is an internal engine component that bridges the gap between disparate physical file formats and the unified virtual filesystem exposed to the game code.

## Quick Start Example

As an engine developer, you may need to write interceptors for new formats. While the specific classes vary, the core mechanism involves registering a mount provider:

```csharp
using Sandbox.Mounting;
using System.Threading.Tasks;

// New mount providers derive from BaseGameMount.
// See engine/Mounting/Sandbox.Mounting/MountedGame/BaseGameMount.cs.
public sealed class CustomEngineMount : BaseGameMount
{
    public override string Ident => "custom";
    public override string Title => "Custom Engine";

    protected override void Initialize( InitializeContext context )
    {
        // Detect the game's install on disk and set IsInstalled.
    }

    protected override Task Mount( MountContext context )
    {
        // Register your assets via context. Set IsMounted = true on success.
        return Task.CompletedTask;
    }
}
```

## The Process

When a mount occurs, the system scans the target directory or archive. It doesn't physically convert all files immediately; instead, it registers interceptors within the `Sandbox.Mounting` namespace.

When the game attempts to load a legacy material, the mounting system intercepts the request, reads the old format, generates the modern Source 2 equivalent in memory, and passes it to the renderer. This avoids needing massive gigabytes of duplicate, converted assets on disk.

## Extensibility & Patterns

This system is generally not extensible from user-level C# (Game Developers). It requires low-level interaction with the engine's asset loading pipeline and is tightly coupled to `Sandbox.Engine`. 

## Troubleshooting
:::warning
- **Interceptor Conflicts:** If two mounts intercept the exact same file extension, the last registered or highest priority mount wins. This can cause confusing bugs where an asset seems to load from the wrong game.
- **Legacy Format Parsing Failures:** The parsers for GoldSrc, Quake, and NS2 rely on exact byte layouts. Modded or corrupted legacy files may cause unhandled exceptions during the read process, aborting the mount.
:::

### Diagnosing Mount Resolution Failures
When diagnosing issues where an asset fails to load or incorrect data is fetched, you need to trace the resolution path through the Virtual File System (VFS). Here are realistic examples of diagnosing failures:

**Example 1: The "Wrong Asset" Bug (Precedence Failure)**
*   **Symptom:** A modder replaced `models/player.vmdl` in their addon, but the engine is still loading the base game's model.
*   **Diagnosis:** The base game's mount was registered *after* the addon mount, or with a higher priority flag.
*   **Action:** Use the Developer Console (`F1`) and execute `mount_list`. The output displays the active mounts in descending priority order. If the base game is higher in the list, you must adjust the `MountHost.Initialize()` precedence or ensure the addon's `MountConfig` explicitly requests an override priority.

**Example 2: The "Silent Fallback" Bug (Interceptor Failure)**
*   **Symptom:** You registered a `CustomFileInterceptor` for `.customext`, but requesting `data.customext` returns `FileNotFoundException` or falls back to an empty byte stream, without any obvious crash.
*   **Diagnosis:** The request reached `FileSystem.OpenRead`, but your interceptor either threw a caught exception internally or returned `null`.
*   **Action:** Enable verbose VFS logging (e.g., `log_vfs_verbose 1` if available via ConVar, or attach a debugger to `Sandbox.Engine`). Set a breakpoint inside your interceptor's `Open()` method. Often, this happens because the interceptor attempts to read a header (like a magic number) that doesn't match the file, silently aborting the read process.

**Example 3: The "Memory Leak on Disconnect" Bug (Unmounting Failure)**
*   **Symptom:** After joining and leaving several servers that mount custom assets, RAM usage balloons, eventually causing an out-of-memory crash.
*   **Diagnosis:** The `MountHost` was torn down, but the `CustomFileInterceptor` cached large `byte[]` buffers in static dictionaries, preventing the Garbage Collector from freeing the memory.
*   **Action:** Take a managed memory snapshot using the built-in Profiler (View -> Profiler) before and after unmounting. Look for lingering instances of `MountedAsset` or byte arrays holding your custom asset signatures. Ensure your interceptors implement `IDisposable` and explicitly clear their internal caches during `MountHost.Dispose()`.

**Example 4: The "Missing Texture" Bug (Case-Sensitivity in Legacy Mounts)**
*   **Symptom:** A legacy GoldSrc map mounts correctly, but specific textures display as the purple/black missing material checkerboard, even though the `.wad` file contains them.
*   **Diagnosis:** Legacy file systems (especially when unpacked on Linux) can exhibit case-sensitivity issues. The engine requests `textures/MyTexture.vtex` (or the map inherently requests the uppercase version), but the VFS or interceptor is strictly searching for `mytexture`.
*   **Action:** When writing or debugging legacy mount interceptors (like `Sandbox.Mounting.GoldSrc`), ensure that filename hashing and string lookups are explicitly case-insensitive (e.g., using `StringComparison.OrdinalIgnoreCase`). Verify by querying the VFS for all files in the directory and checking the exact casing of the extracted assets.

**Example 5: The "Silent Ignore" Bug (Archive Parsing Failure)**
*   **Symptom:** A Quake `pak0.pak` file is placed correctly, but none of its contents are available to the VFS.
*   **Diagnosis:** The Quake `.pak` parser expects a strict 12-byte header, specifically verifying the `PACK` magic signature. If the header mismatches or the index length is malformed, the `Pack` instance internally marks itself invalid and silently skips registering the files. This is a common quirk with modified or custom Quake assets that don't perfectly align with the original ID software archive specification.
*   **Action:** When diagnosing missing archived assets, verify the archive's internal integrity. Attach a debugger to the `Pack` constructor in `Sandbox.Mounting.Quake` to ensure the 12-byte header parse completes successfully.

**Example 6: The "Unsupported Extension" Skip (Mapping Failure)**
*   **Symptom:** In a Natural Selection 2 mount, Spark engine files like `.gui` or custom formats simply don't resolve in the VFS, while `.model` and `.material` work perfectly.
*   **Diagnosis:** Interceptors selectively process files based on a predefined dictionary (like `FileTypes` in NS2's `GameMount.cs`). If an extension is not registered, the `Mount` task silently ignores the file rather than parsing it as raw bytes. This quirk occurs because the Spark Engine utilizes a much broader, highly specific set of extensions compared to typical Source games, making 1-to-1 conversion difficult without explicitly defined interceptors. If the extension mapping doesn't cleanly translate the raw bytes into a modern Source 2 construct, the engine simply drops the asset to prevent runtime pipeline crashes.
*   **Action:** If a specialized asset is missing, confirm that the file extension is mapped in the mount's `Initialize` or `Mount` routines, and that a loader logic specifically supports its translation into a Source 2 construct. Verify the internal dictionaries inside `Sandbox.Mounting.NS2` are acknowledging the specific `.ext` being requested.

### Specific Quirks: Remote, Quake, and NS2 Mounts

While legacy formats like GoldSrc present significant challenges around signature parsing, newer interceptors or network-based mounts expose their own architectural quirks.

**Example 7: Remote Mounting Quirk (Timeouts & Sync Reads)**
*   **Symptom:** When attempting to mount an asset from a remote HTTP source (e.g., streaming a map from a community server), the engine hangs for several seconds and then falls back to a missing asset, even though the URL is valid.
*   **Diagnosis:** Remote mounts often rely on asynchronous network streams. If the main thread's `FileSystem.OpenRead` blocks waiting for the initial byte stream to buffer, and the connection is slow, the VFS might hit a strict internal timeout to prevent a complete editor or client freeze. The engine fundamentally expects `FileSystem.OpenRead` to return a stream almost instantly.
*   **Action:** Ensure that any custom network interceptors implement proper async pre-fetching. Specifically, avoid synchronous `.Result` or `.Wait()` calls on network streams inside your `Open()` logic. Instead, structure your `MountHost` pipeline to download the asset into a temporary local cache *before* it gets mapped. 
```csharp
// BAD: Synchronous block inside Mount execution
public override Stream Open( string filePath ) {
    return _httpClient.GetStreamAsync( url ).Result; // Can freeze the VFS thread!
}

// GOOD: Pre-fetch and stream from local cache
public async Task PreFetchAssetAsync( string url, string cachePath ) {
    var bytes = await _httpClient.GetByteArrayAsync( url );
    await File.WriteAllBytesAsync( cachePath, bytes );
    // Map 'cachePath' to the VFS interceptor only AFTER download completes.
}
```

**Example 8: Quake Palette Resolution (Color Corruption)**
*   **Symptom:** You mount a Quake 1 `.pak` file, and the models load, but the textures look like random, garbled noise or completely flat grayscale blocks.
*   **Diagnosis:** Quake 1 textures are paletted (8-bit indices) rather than modern RGBA. The `Sandbox.Mounting.Quake` interceptor must locate a specific `palette.lmp` file (usually within the same `pak0.pak` or `gfx/` directory) to decode the raw bytes into a standard RGBA buffer for the Source 2 material generator. If the palette is missing or fails to parse, the interceptor fallback defaults to mapping the indices directly to a grayscale buffer or produces garbage output depending on the exact header state.
*   **Action:** When debugging Quake mounts, verify the VFS can resolve `gfx/palette.lmp` *before* attempting to load models or maps. The interceptor's initialization phase usually logs a warning if the palette is unavailable. Ensure your `pak0.pak` hasn't had its `gfx/` directory stripped by compression tools.

**Example 9: Natural Selection 2 (NS2) Scale Mismatch**
*   **Symptom:** NS2 assets mount successfully, but player models and props appear comically large or clip through the floor compared to native s&box assets.
*   **Diagnosis:** The Spark engine used different unit scales and coordinate system handedness compared to Source 2. The `Sandbox.Mounting.NS2` interceptor applies a hardcoded scale transformation matrix during the conversion of the mesh vertices and skeleton to bring it into Source 2's inch-based system. If a custom or modded NS2 model bypasses the standard export pipeline or uses unconventional coordinates, this scaling might be applied incorrectly.
*   **Action:** Inspect the interceptor's mesh generation code (typically where it constructs the `CModel` representation inside the NS2 parser). If working with heavily modified NS2 assets, you may need to introduce configuration variables to the `MountConfig` to manually override the import scale factor applied during the vertex buffer generation.

1. **Check the Virtual Filesystem Routing:**
   To determine if a request reached the VFS versus an interceptor, monitor `FileSystem.OpenRead` calls using internal logging. A successful interceptor triggers `Sandbox.Mounting.MountedAsset.Open`.
2. **Review Internal Logging:**
   If the interceptor failed, look for log warnings about unsupported magic numbers (e.g., a mismatched `.wad` file signature). These warnings indicate a **Legacy Format Parsing Failure**.
3. **Verify Mount Registration Order:**
   Use the Developer Console (`F1`) and execute diagnostic commands (if available) to dump active mounts (`mount_list`). Verify that the addon you are testing has a higher precedence than standard fallbacks.

## Related Pages
- [GoldSrc Mounting](../systems/mounting/goldsrc.md)
