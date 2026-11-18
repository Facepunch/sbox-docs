---
title: "Mounts for Dedicated Servers"
icon: "🎮"
created: 2026-11-18
updated: 2026-11-18
---

# Mounts for Dedicated Servers

Dedicated Servers can access content from installed Steam games via [Game Mounts](/game-mounts/index.md).

Unlike in the s&box client, however, mounts on servers require manual configuration to enable.

## Mount Config

Mounts are configured in `config/mounts.json`. Each key is the ident of a mount (for example `gmod`, `quake` or `cstrike`), and its value holds the settings for that mount.

To activate a mount, set `Enabled` to `true` and set `Path` to the game's installation directory on the server. The server will attempt to load its content when both are set.

```json
{
  "gmod": {
    "Enabled": true,
    "Path": "C:/path/to/gmod"
  }
}
```