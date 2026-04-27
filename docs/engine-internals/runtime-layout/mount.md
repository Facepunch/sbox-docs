---
title: "Mount Systems"
icon: "🗜️"
sources:
  - game/mount
created: 2026-04-27
updated: 2026-04-27
---

# Mount Systems

The `mount` directory (`game/mount/`) contains built-in configurations and assets used by the engine to mount content from other classic Source Engine games.

## Included Mount Types

The `game/mount/` directory contains predefined configurations for specific engine eras:

1. **GoldSrc (`game/mount/goldsrc/`)**: Configuration to mount and read assets from Half-Life 1 era games. This includes translating old `.wad` texture files and skeletal animation `.mdl` formats.
2. **Quake (`game/mount/quake/`)**: Configuration to mount original Quake assets (BSP formats and palette conversions).
3. **NS2 (`game/mount/ns2/`)**: Configuration for mounting assets from Natural Selection 2.

## How it works

When a game or addon requests an asset (e.g., `models/player.mdl`), the engine searches its standard virtual filesystem. If a legacy mount is active and the asset is found within a mounted legacy game folder, the engine's internal `Sandbox.Mounting` namespace code intercepts the request. It then translates the legacy asset into a format the Source 2 engine can understand in memory, before handing it back to your C# code or the renderer.

## Usage in Projects

You typically configure mounts within your project's `.sbproj` file under a `"mounts"` or `"dependencies"` section if you require specific legacy games to be installed on the user's machine to play your game.

## Troubleshooting

:::warning "Asset not found when mounting!"
If you are trying to mount a legacy game like Counter-Strike: Source, the user **must** own and have the game installed on Steam. s&box does not ship with the assets from other games. It only ships with the *ability* to read them.
:::

:::warning "Materials look wrong on ported models"
Because older engines used a fundamentally different rendering model (specular/gloss rather than PBR roughness/metallic), the automatic translation is not always perfect. You may need to manually create material overrides for specific assets to look correct under Source 2 lighting.
:::

## Related Pages
- [Mounting System](mounting.md)
