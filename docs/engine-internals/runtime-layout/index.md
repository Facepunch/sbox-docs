---
title: Runtime Layout & Architecture
sources:
  - engine/Launcher
  - game/bin
  - game/core
  - game/addons
  - game/mount
  - game/samples
  - game/templates
updated: 2026-04-25
created: 2026-04-27
---

# Runtime Layout & Architecture

Understanding the physical layout and the runtime mapping of the s&box engine is crucial for engine developers working on build pipelines, mounting interceptors, or the underlying .NET hosting environment.

This section covers the underlying architecture of the s&box engine and the game directory layout from a systems perspective.

## Engine Layout Overview

The project is broadly split into two main areas:

*   **`engine/`**: Contains the core s&box engine code and systems (e.g., `Sandbox.Engine`, `Sandbox.System`, etc.).
*   **`game/`**: Contains the runtime files, binaries, core game logic, and default mounted content formats.

## In This Section

*   [Game Architecture](game-core.md) - Details how `Sandbox.Engine` interceptors resolve the base `game/core` footprint.
*   [Mounting System](mounting.md) - Internal implementation details of `MountHost` and VFS precedence mapping.
*   [Mount](mount.md) - Specifics on the `game/mount` runtime execution logic.
*   [Engine Launcher](launcher.md) - Details about `engine/Launcher` startup.
*   [.NET Core Hosting](launcher/netcore-hosting.md) - Details on how the CoreCLR is brought up inside the native process.

## Related Pages
* [Mounting Internals](../mounting.md)
* [Sandbox.Engine](../sandbox-engine/index.md)
