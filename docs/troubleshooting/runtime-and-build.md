---
title: Runtime & Build
icon: "🚀"
created: 2026-04-25
updated: 2026-04-25
---

# Runtime & Build

Standalone export issues, missing assets in shipped builds, dedicated server bind problems, mounting and addon dependencies.

## Standalone export

### "Build Failed: Unauthorized"

You're not logged in to the Editor with a valid developer account.

- **Fix:** log into the Editor with your sbox.game account first, then re-run export.

**Deep dive:** [Exporting Standalone](../publishing/exporting-standalone.md).

### "Standalone build is missing assets"

Assets weren't included in the build configuration. Standalone doesn't have access to the editor-time mount.

- **Fix:** ensure all assets are referenced (loaded by code or scene) so the build pipeline includes them.
- **Check:** dependencies in `.sbproj` are correct — addons your game relies on must be listed.

**Deep dive:** [Standalone](../systems/platform/standalone.md).

### "Do not distribute yet warning"

Local exports work for testing but the standalone license is still being finalized — don't publicly redistribute the standalone yet.

- **Fix:** test locally; wait for the official standalone distribution path.

## Publishing addons / packages

### "Package already exists" error

Your Organization+Ident combination is already taken on sbox.game.

- **Fix:** choose a unique Ident (or move to your own Organization).

**Deep dive:** [Publishing](../publishing/index.md).

### "I can't make a C# addon"

Addon Projects don't support C# code currently. Only ActionGraphs.

- **Workaround:** put logic in a Game Project that consumes the Addon, or wait for the feature.
- **Source:** [Addon Project](../getting-started/project-types/addon-project.md), [Publishing Addons](../publishing/publishing-addons.md).

### "Addon doesn't show up after a settings change"

Changing the target game requires an Editor restart.

- **Fix:** restart the s&box Editor after changing target game.

### "I can't play my addon directly"

Addons aren't directly playable. They need a Game Project as a host.

- **Fix:** create a Game project, add your addon as a dependency, then play the Game project.

**Deep dive:** [Addon Minimal](../build-games/samples-templates/addon-minimal.md).

## Mounting and dependencies

### "Missing Dependency / Addon not loading"

Spelling in `.sbproj` is wrong, or the addon hasn't been published yet.

- **Check:** dependency Ident is exact (`facepunch.citizen`, not `Facepunch.Citizen`).
- **Check:** the addon is publicly available, or it's a local addon that exists on disk.

**Deep dive:** [Addons](../engine-internals/addons/index.md), [Mounting](../systems/mounting.md).

### "Asset paths are pink/black checkers in shipped build"

Asset wasn't packaged because nothing referenced it, or it depends on a mount the player doesn't have.

- **Check:** the asset is referenced (in a scene, prefab, or code) so it gets included.
- **Check:** if the asset is from a mounted game (CSS, HL2), the player must own that game on Steam.

**Deep dive:** [Mounting Internals](../engine-internals/mounting.md), [Mount](../engine-internals/runtime-layout/mount.md).

### "File not found from a legacy Source mount"

Player doesn't own/installed that legacy game.

- **Fix:** check `IsInstalled` before loading legacy assets and provide a fallback.

**Deep dive:** [GoldSrc Mounting](../systems/mounting/goldsrc.md), [NS2 Mounting](../systems/mounting/ns2.md), [Quake Mounting](../systems/mounting/quake.md).

### "Two mounts intercept the same extension — wrong asset loads"

Mount priority decides which wins; the last registered or highest-priority mount handles the file.

- **Fix:** set explicit mount priority, or namespace your assets so they don't collide.

## Dedicated servers

### "Players can't connect over the internet"

Port not forwarded. Default is **27015** (UDP and TCP).

- **Fix:** open 27015 on the router and the host firewall.

**Deep dive:** [Dedicated Servers](../systems/networking-multiplayer/dedicated-servers/index.md).

### "Game works in editor, breaks on dedicated"

Editor runs as host+client (you have both roles); dedicated has no client.

- **Fix:** guard authority checks with `Networking.IsHost`, and client-only code with `if ( !Application.IsHeadless )`.

### "Server is running a different version"

Server's compiled addon hash mismatches the client's.

- **Fix:** make sure the server pulled the latest code and recompiled before clients try to join.

### "Client timed out"

Server's main thread blocked too long (e.g., a giant `while` loop in a Component) and dropped keep-alives.

- **Fix:** profile and break up long-running synchronous work. Use async/`Task.Yield()` for chunked work.

**Deep dive:** [Server Architecture](../systems/networking-multiplayer/server-architecture.md), [Sandbox.Server](../engine-internals/sandbox-server.md).

## Steam and platform

### "Steam isn't initializing in my Standalone build"

`SteamClient.IsValid` is false — Steam isn't running, or `steam_appid.txt` is wrong/missing.

- **Fix:** ensure Steam is running, the user is logged in, and the AppID file is correct.

**Deep dive:** [Steam](../systems/platform/steam.md), [Platform](../systems/platform/index.md).

### "Achievement isn't unlocking"

Identifier mismatch (case-sensitive) or achievement not defined on backend.

- **Check:** the achievement Ident matches what's defined on sbox.game's dashboard.

**Deep dive:** [Achievements](../systems/services/achievements.md).

### "Stat / Achievement not found"

Same family — undefined on the dashboard.

- **Fix:** define the stat/achievement on sbox.game's project dashboard before referring to it in code.

**Deep dive:** [Services](../systems/services/index.md).

### "HTTP 401 Unauthorized / Not logged in"

Steam isn't running, or your developer token expired.

- **Fix:** start Steam, ensure you're logged in. Re-link your project if a token expired.

**Deep dive:** [Backend Architecture](../systems/services/architecture/backend.md).

### "OpenXR / VR.IsActive is false"

User's OpenXR runtime isn't set correctly. SteamVR users need SteamVR set as the active OpenXR runtime in SteamVR Developer settings.

- **Fix:** confirm the active OpenXR runtime in the user's VR system.

**Deep dive:** [VR Platform](../systems/platform/vr.md).

## Filesystem and storage

### "Access Denied / System.IO Exceptions"

Direct .NET I/O is sandboxed.

- **Fix:** use `Sandbox.FileSystem.Data` for writable user storage, `Sandbox.FileSystem.Mounted` for read-only assets.

**Deep dive:** [File System](../systems/file-system/index.md).

### "Variables aren't saving to JSON"

Default `WriteJson` only serializes **properties**, not fields.

- **Fix:** `public int Score { get; set; }` (property), not `public int Score;` (field).

**Deep dive:** [Save and Load Data](../how-to/save-and-load.md).

### "I can't write to a downloaded Workshop entry"

Installed Workshop content is **read-only**.

- **Fix:** copy data into your own writable Storage entry if you need to mutate it.

**Deep dive:** [Storage UGC](../systems/file-system/storage-ugc.md).

### "Invalid storage type"

`Storage.CreateEntry( type )` expects 1-16 characters, **letters only** (a-z, A-Z).

- **Fix:** rename to letters-only.

## System requirements

### "CPU needs to support AVX instructions"

User's CPU is too old. AVX is a hard requirement and the engine refuses to launch.

- **Fix:** there's no workaround — it's a minimum hardware requirement.

**Deep dive:** [System Requirements](../getting-started/system-requirements.md), [AppSystem](../engine-internals/runtime-layout/appsystem.md).

### "Poor performance on a laptop"

Laptop is using integrated graphics instead of the dedicated GPU.

- **Fix:** in Windows Graphics Settings (or Nvidia Control Panel), force s&box to use the discrete GPU. Plug in the laptop.

### "Integrated Intel graphics not supported"

Hard limitation. AMD APUs (Ryzen integrated) work but performance varies.

- **Fix:** discrete GPU recommended.

## Engine console errors

### "Network String Table limit exceeded"

Procedural generation is creating too many unique networked strings.

- **Fix:** audit code that creates networked names/IDs in a loop. Reuse strings or use IDs.

**Deep dive:** [Game Instance](../engine-internals/runtime-layout/game-architecture/game-instance.md).

### "MethodNotFound after engine update"

Native engine binding changed, your managed code wasn't regenerated.

- **Fix:** if you're working on engine source, run `InteropGen` to regenerate C# bindings.

**Deep dive:** [InteropGen](../engine-internals/tools/interopgen.md).

### "Pink and Black Textures from core"

Material path invalid, or core wasn't mounted.

- **Don't:** modify files in `game/core/` — engine updates wipe them.
- **Fix:** copy the core asset into your own addon and modify the copy. Reference the copy.

**Deep dive:** [Game Core](../engine-internals/runtime-layout/game-core.md).

## Related Pages

- [Networking](./networking.md)
- [Hotload & Compile](./hotload-and-compile.md)
- [Editor](./editor.md)
