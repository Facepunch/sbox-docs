---
title: "Game Mounts"
icon: "🎮"
created: 2026-04-19
updated: 2026-04-19
---

# Game Mounts

Game Mounts let users play with assets from other games they have installed inside s&box. Models, textures, materials, and sounds are converted on-the-fly - no manual importing or file conversion required.

Mounted assets are runtime only, it's not possible to publish or ship them with games. Players need to own and install the original games on Steam to play with the assets.

## How It Works

Each mount is a plugin that knows how to find and convert assets from a specific game. When you enable a mount, it detects the game's install directory via Steam, scans for assets, and converts them into s&box compatible assets all at runtime. Mounted assets are accessed via `mount://` paths and lazy-loaded on demand.

Mounts are toggled in the **Asset Browwser** or used in the **Sandbox Spawn Menu**. The system is extensible - anyone can write a new mount and contribute it to s&box via pull request.
