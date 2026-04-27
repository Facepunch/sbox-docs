---
title: "Status"
icon: "🟠"
created: 2024-01-19
updated: 2025-06-15
sources:
  - engine/Sandbox.Engine/Core/EngineLoop.cs
---

# Status

As you play with developing in s&box, you will notice some things are weird, missing, or suck. We are aware. 
Because the engine is actively developed, the API surface changes and new systems are introduced regularly.

## Quick Working Code Example: Feature Checking

If you are writing complex systems that might break based on engine updates, you can check the engine's build info at runtime (though usually the compiler will stop you if APIs have been completely removed).

```csharp
using Sandbox;

public sealed class EngineVersionCheck : Component
{
    protected override void OnStart()
    {
        // Engine.Version usually contains the build identifier
        Log.Info( $"Running on s&box Engine Version: {Engine.Version}" );
    }
}
```

## Current Feature Status

Here's the current status of key features and missing functionality you'll run into.

| Name | Description |
|:-----|-------------|
| Audio | 🟢 Can play sounds, music, voice chat, can play them positionally or 2D<br />🟢 Audio processing, effects system<br />🟢 Lip sync<br />🟠 No auto voice ducking |
| Multiplayer | 🟢 Can create server, join server, sync vars, gameobjects, rpcs<br />🟢 Dedicated server support<br />🟠 No error handling, permissions |
| UI   | 🟢 Razor and Panel class UI, full style sheets, transitions, transforms, animation, world panels |
| Scenes | 🟢 Can create scene, gameobject, components, play, stop |
| Controller | 🟢 Controller input<br />🟠 UI navigation with virtual cursor, missing element navigation, on-screen keyboard |
| Navigation | 🟢 Recast navmesh, agents, pathfinding, links |
| Platforms | 🟢 Windows<br />🟠 Is likely to expand in the future, just not a priority |
| Hammer/Maps | 🟢 Create + Load maps, brushwork, props, lights, select + launch map in game🟢 Can use GameObjects |
| VR   | 🟢 Easy to develop for, built-in components<br />🟠 Accessibility, user experience needs work, needs more testing |
| Particles | 🟢 Create particle effects, emitters, sprite rendering, model rendering, trails, lights, velocity, collision<br />🟢 Totally moddable and extendable<br />🟠 No built in line renderer, motion vectors, normals |
| Standalone | 🟠 Working on export to standalone version license. Exporter works. |
| Physics | 🟢 Full 3D and 2D physics, great performance + stability |
| Editor | 🟢 Scene editor, editor tools, inspector, editor apps, key re-binding |
| 2D   | 🟢 2D physics, ortho camera<br />🟠 Basic sprite support<br />🔴 No 2d editor mode |
| GameResources | 🟢 Custom game resource types, with inspector editing<br />🟠 No binary writing support yet |
| Terrain | 🟠 Early development, basic editing<br />🔴 No grass/detail models yet |
| Animation | 🟢 Valve's animgraph system, events, IK, animated ragdolls, morphs<br />🟠 Some advanced features currently rely on an undocumented format |
| PostProcessing | 🟢 Shader support, camera-based post-process base component  |

## Troubleshooting

:::warning Dealing with Missing Features
- **"I need X feature but it says 🔴 or 🟠."**
  Often, you can implement missing high-level features yourself in C#. Because s&box provides deep access via C#, you aren't strictly blocked. For instance, if a specific 2D Editor mode is missing, you can build your own Editor tool using the `Editor` namespace.
- **Breaking Changes:**
  As features move from 🔴 to 🟢, the API will change. Keep an eye on the official changelogs and forum announcements. If your project suddenly fails to compile after an update, a method you were using was likely renamed or moved.
:::

## Related Pages
- [Reporting Errors](reporting-errors.md)
