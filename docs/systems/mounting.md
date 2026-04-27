---
title: Mounting System
icon: "📂"
updated: 2026-04-25
created: 2026-04-27
---

# Mounting System

The mounting system has two related but distinct surfaces. Pick the one that matches what you're trying to do:

| You want to... | Go to |
|---|---|
| Use assets from other Steam games (GoldSrc, Quake, NS2) | [Game Mounts](mounting/index.md) |
| Write a new mount for a game we don't support yet | [Creating Mounts](mounting/creating-mounts.md) |
| Understand how the engine layers addons into a single virtual filesystem | [Mounting (Engine Concepts)](../engine-concepts/mounting.md) |
| Look up the engine-side `MountHost`/`BaseGameMount` API | [Mounting Internals](../engine-internals/mounting.md) |

Most game devs want the first row. The rest is for people writing engine extensions or trying to understand the boot-time filesystem layering.

## What "mount" can mean

The word "mount" appears in two contexts and that's confusing:

- **Game mounts** — plugins that load assets from external games (Quake, GoldSrc, NS2). Toggled in the Asset Browser. Assets accessed at `mount://<ident>/<path>`. **This is what game devs usually mean.**
- **Filesystem mounts** — engine-internal layering. Each addon (`facepunch.base`, `facepunch.citizen`, your project) is a filesystem layered into a single `FileSystem.Mounted`. You don't manage these directly; they're set up by the package manager. See [Engine Concepts → Mounting](../engine-concepts/mounting.md).

The two systems share machinery (the virtual filesystem layered stack) but the user-facing surface is different.
