---
title: Troubleshooting
icon: "🔨"
created: 2026-04-27
updated: 2026-04-27
---

# Troubleshooting for Players

If you are encountering issues while trying to play games in s&box, this guide covers common problems and how to resolve them.

## Common Issues & Fixes

### Missing Assets (Pink/Black Checkers or ERROR signs)

**The Problem:** You see giant red `ERROR` models, or textures are covered in pink and black checkerboards.

**The Cause:** The engine cannot find the visual assets needed to render an object. This usually means either your download failed midway, or the game developer forgot to include the asset in their game package.

**How to Fix:**
1. Reconnect to the server. This often forces the engine to retry downloading missing files.
2. If it persists across multiple servers running the *same* game mode, the developer likely broke an update. You will have to wait for them to fix it.
3. Verify your base engine files in Steam (Right-click s&box -> Properties -> Local Files -> Verify integrity).

### Game Crashes to Desktop

**The Problem:** s&box completely closes without warning while playing.

**The Cause:** This can be caused by running out of memory, outdated graphics drivers, or a bug in the specific custom game you are playing.

**How to Fix:**
1. **Update Graphics Drivers:** Ensure your NVIDIA or AMD drivers are up to date.
2. **Lower Settings:** If you are running out of VRAM, lower texture quality in the settings menu.
3. **Isolate the Cause:** Does the engine *only* crash when playing a specific game mode? If yes, the bug is in that custom code, not the engine. Report it to the game's creator.

### "Connection Failed" or "Server Not Responding"

**The Problem:** You cannot join any multiplayer servers.

**The Cause:** Network routing issues, firewalls, or the server crashed.

**How to Fix:**
1. Ensure `sbox.exe` is allowed through your Windows Firewall.
2. If hosting a game for friends, ensure you aren't hidden behind a strict NAT (though Steam Relay usually handles this automatically now).

## Stuttering and Performance

### Shader Compilation Stutter

When you first join a new game mode, you may experience brief, heavy stutters as you look around the map.

**This is normal.** The engine is compiling shaders for the first time. As you continue playing, the cache builds up and the stuttering will stop. The next time you launch that game, it will be smooth from the start.

### High CPU Usage

Because s&box games run on user-generated code, some poorly optimized games will cause low framerates regardless of your hardware. If a specific game mode lags while others run fine, it is an issue with that game's code.

## Reporting Bugs

* **If a specific game mode is broken:** Try to contact the creator of that game mode (often via their Discord, linked on the game's menu page).
* **If the engine itself is broken (e.g., the Main Menu won't load, every game crashes):** Check the [s&box forums](https://sbox.game/f/) or the [public GitHub issue tracker](https://github.com/Facepunch/sbox-public/issues).
