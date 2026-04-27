---
title: "Tutorials"
icon: "🎓"
sources:
  - game/samples
updated: 2026-04-25
created: 2026-04-27
---

# Tutorials

Tutorials are *learning-oriented* — guided walks that teach you how things work by building something concrete. If you want a quick code snippet for a known task, use [Cookbook](../how-to/index.md) instead.

## Start here

If you've never opened s&box before:

1. [Your First Game](your-first-game.md) — build a complete tiny game from scratch. Sets your expectations for what an s&box project actually looks like.
2. [Your First Component](your-first-component.md) — line-by-line walkthrough of writing a `Component`. The most fundamental thing you'll do as an s&box dev.

These two assume zero prior knowledge of the engine.

## When you're past the basics

Pick whichever matches what you're trying to build:

| Tutorial | What you build | Topics |
|---|---|---|
| [Build a Third-Person Controller](build-a-third-person-controller.md) | A character with run, jump, and a follow camera | `PlayerController`, camera config, extending built-in components |
| [Build a NavMesh AI](build-a-navmesh-ai.md) | An NPC that patrols and chases the player | `NavMeshAgent`, simple state machines, debug overlays |
| [Build a Networked Lobby](networking-basics.md) | A two-player local-test lobby with synced player health | `NetworkSpawn`, `[Sync]`, `[Rpc.Broadcast]`, `IsProxy`, `Connection` |
| [Build a User Interface](build-a-ui.md) | A HUD panel with a health bar and a button | Razor `PanelComponent`, `[Bind]`, conditional rendering, styling |

You don't need to do these in any order. Each is self-contained.

## What you'll need before starting

- s&box installed and a project open (see [Getting Started](../getting-started/index.md) if you don't have one yet)
- An external code editor — Visual Studio, Rider, or VS Code (s&box doesn't ship with one)
- The basics of C# — classes, methods, fields, properties. You don't need to know `async/await` or LINQ; the tutorials avoid them.

If you don't have those yet, do [Getting Started](../getting-started/index.md) first — it's quick.

## Stuck on a tutorial

The most common issue: **you saved the C# file but the editor still shows the old behaviour.** Almost always one of:

- There's a compile error in the editor console (red text). Hotload pauses entirely until you fix it.
- You didn't actually save the file. Check the title bar of your IDE — unsaved files have a `*` or dot.
- You attached the component to a GameObject other than the one you're testing on.

Past that, the [Troubleshooting](../troubleshooting/index.md) section is organized by symptom — search for what you're seeing.

## Where to go after these

- [Sample Projects](../build-games/samples-templates/index.md) — full working games you can read and modify.
- [Cookbook](../how-to/index.md) — recipes for specific tasks once you know the basics.
- [Engine Systems](../systems/index.md) — deep dives by topic.
