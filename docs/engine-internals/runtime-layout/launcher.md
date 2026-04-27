---
title: Engine Launcher
sources:
  - engine/Launcher
updated: 2026-04-25
created: 2026-04-27
---

# Engine Launcher

> **See also:** [Launcher Internals](../launcher.md) — engine-developer deep dive on the bootstrap flow. [Launcher (Engine Concepts)](../../engine-concepts/launcher.md) — high-level conceptual overview. [.NET Core Hosting](launcher/netcore-hosting.md) — child page on the .NET host.

The Engine Launcher (`engine/Launcher`) is the entry point for the s&box application. It is responsible for initializing the core engine systems, setting up the environment, and starting either the game client, the dedicated server, or the development editor.

## Responsibilities of the Launcher

When you run s&box (via Steam or by executing the binaries in `game/bin/`), the Launcher performs several critical tasks:

1.  **Environment Initialization:** Sets up logging, crash reporting, and base file paths.
2.  **Mounting Core Filesystems:** Mounts the `engine/` and `game/core/` directories so foundational assets and libraries are available.
3.  **Bootstrapping the Engine:** Initializes the graphics API (Vulkan, DX11), audio system, physics engine, and networking layer.
4.  **Mode Selection:** Determines which mode to run in based on command-line arguments:
    *   **Client:** Starts the standard game client.
    *   **Server (`sbox-server`):** Starts a headless dedicated server.
    *   **Editor (`sbox-dev`):** Starts the s&box development editor tools.
5.  **Loading the Menu Addon:** Once the core engine is running, the launcher typically mounts and loads the `menu` addon to display the main interface.

## Runtime Configurations

You might notice several `.runtimeconfig.json` files in the `game/` directory (e.g., `sbox-dev.runtimeconfig.json`, `sbox-server.runtimeconfig.json`). These files configure the .NET runtime environment for the different modes the launcher can initialize, ensuring the correct dependencies and garbage collection settings are used for each context.

Understanding the Launcher's role helps clarify the boot process of s&box and how different parts of the engine are brought online.
