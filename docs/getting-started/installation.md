---
title: "Download & Install"
icon: "📥"
created: 2026-04-09
updated: 2026-04-27
sources:
  - engine/Launcher/SboxDev/
---

# Download & Install

s&box is available to everyone on Steam.

## Install

![Launch the editor from your Steam library](./images/launch-the-editor-from-your-steam-library.png)

[Click here to open s&box on Steam](steam://run/2129370), or:

1. Open Steam
2. Go to the Library tab
3. Search for **s&box**
4. Install both the **game** and the **editor** apps

## Launch

- **Through Steam:** The easiest way — just hit Play in your library.
- **Via `sbox-dev.exe` shortcut:** Useful for passing custom command-line arguments (e.g. `-project C:/MyGame/MyGame.sbproj` to open a project directly).
- **Open `.sbproj` directly:** Associate `.sbproj` with the engine so double-clicking your project file opens the editor instantly.

## First launch tips

- The first launch (and first time opening a new project) will trigger a shader compile pass that spikes CPU. Subsequent launches are fast.
- Configure your IDE under **Edit → Editor Preferences → Code** — Visual Studio, Rider, and VS Code all work.
- You can run multiple instances of `sbox-dev.exe` for local multiplayer testing; launch each from a separate project directory.

## Troubleshooting

:::warning
- **Doesn't launch from Steam:** Verify game files in Steam, or run `sbox-dev.exe` directly as administrator.
- **Antivirus flagging it:** Add your s&box install folder as an exception. The hotload compiler (`sbox-dev.exe`, `dotnet.exe`) triggers some AV heuristics.
- **Missing .NET dependencies:** Install the latest .NET SDK if you're using external build scripts.
:::

## Related Pages

* [System Requirements](system-requirements.md)
* [Your First Project](first-project.md)
