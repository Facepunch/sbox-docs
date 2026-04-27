---
title: "FAQ"
icon: "ℹ️"
created: 2024-02-13
updated: 2026-04-27
sources:
  - game/addons/menu/Code/
---

# FAQ

## General

### How do I get it?

s&box is releasing on [Steam](https://store.steampowered.com/app/590830/sbox/) for $20.

Access to the developer preview is now closed — you may or may not keep it post-release depending on play time, development activity, or skins purchased.

### Is it an engine or a platform?

Both. You use the **Engine** (Editor, C#, Scene System, Source 2) to build your game. When you publish, you push it to the **Platform** (sbox.game backend). Players launch the s&box client and the platform dynamically downloads and mounts your game — no manual install needed.

### Are you using Steam Workshop?

No. s&box uses [sbox.game](https://sbox.game) instead. Games and addons are streamed on demand when a player joins — similar to how YouTube doesn't ask you to "install" a video.

### Why not Unity or Unreal?

We're sentimental about Source. Source 2 has a great renderer and brings the engine up to date with tools like ModelDoc and Animgraph. It would have been cheaper to use Unity or UE, but we would have loved it less.

## Monetization

### Can creators make money?

Yes — it's integral. There are currently two paths:

- **Clothing / cosmetics:** Sell items via Steam Workshop and receive a cut.
- **Play Fund:** Make a game and [earn a share of the Play Fund](monetization.md) based on playtime.

### Why isn't my game earning from the Play Fund?

The Play Fund requires your game to meet minimum engagement metrics (unique players, playtime) over a qualifying period. Publishing alone doesn't trigger payment.

### How do I clear downloaded game cache?

Through the s&box Main Menu settings — not through Steam's storage manager, since s&box bypasses the Workshop.

## Related Pages
- [Monetization](monetization.md)
- [Project Types](project-types/index.md)
- [For Players](../for-players/index.md)
