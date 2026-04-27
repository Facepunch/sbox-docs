---
title: "ISceneStartup"
icon: "▶️"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Events/ISceneStartup.cs
---

# ISceneStartup

`ISceneStartup` is a scene-event interface for hooking into the scene's start-up phase. It fires once per scene activation and distinguishes between **host** and **client** stages so you can run setup logic on the right machine. It is primarily intended for `GameObjectSystem`s — most components do not yet exist when the earliest of these callbacks runs.

## Quick Working Example

```csharp
using Sandbox;

public sealed class GameBootstrap : GameObjectSystem<GameBootstrap>, ISceneStartup
{
    public GameBootstrap( Scene scene ) : base( scene ) { }

    void ISceneStartup.OnHostPreInitialize( SceneFile scene )
    {
        // Scene file is selected but not yet loaded. Patch it before objects are spawned.
    }

    void ISceneStartup.OnHostInitialize()
    {
        // Scene is loaded on the host. Spawn host-authoritative objects here.
        Log.Info( "Host scene ready." );
    }

    void ISceneStartup.OnClientInitialize()
    {
        // Local scene is fully loaded (after the host snapshot has been applied for joining clients).
        Log.Info( "Client scene ready." );
    }
}
```

## Interface

```csharp
public interface ISceneStartup : ISceneEvent<ISceneStartup>
{
    /// <summary>Called before the scene is loaded. In game only, on host only.</summary>
    void OnHostPreInitialize( SceneFile scene ) { }

    /// <summary>Called after the scene is loaded. In game only, on the host only.</summary>
    void OnHostInitialize() { }

    /// <summary>
    /// Called in game after the client has loaded the initial scene from the server,
    /// or after OnHostInitialize. This is not called on the dedicated server.
    /// </summary>
    void OnClientInitialize() { }
}
```

All three methods have empty default implementations — implement only the ones you care about.

## When Each Method Fires

| Method | Where | When | Typical use |
|---|---|---|---|
| `OnHostPreInitialize( SceneFile scene )` | Host only, in-game only | Before the chosen scene is loaded — the world is empty. | Modifying the `SceneFile` before objects are instantiated; setting up host-only `GameObjectSystem` state. |
| `OnHostInitialize()` | Host only, in-game only | After the scene has finished loading on the host. | Spawning host-authoritative objects (game manager, world clock, common cameras). |
| `OnClientInitialize()` | Every machine **except** the dedicated server | For the host, immediately after `OnHostInitialize`. For joining clients, after they have received and applied the initial scene snapshot from the server. | Local-only setup: client-side UI, sound, post-processing. |

`ISceneStartup` is dispatched via the scene event bus, so any subscriber that implements it — `GameObjectSystem`s and `Component`s alike — gets the call. Components that exist at scene-load time will receive `OnClientInitialize`, but `OnHostPreInitialize` runs **before any components have been created**, so it's only useful from a `GameObjectSystem`.

## Components vs. GameObjectSystems

`ISceneStartup` is documented in the source as primarily for `GameObjectSystem`s "because components won't have been spawned/created when most of this is invoked." A practical rule of thumb:

- For host-side bootstrap that must run before any object exists → `GameObjectSystem` + `OnHostPreInitialize`.
- For host-side post-load setup → `GameObjectSystem` + `OnHostInitialize`.
- For local "the scene is ready, do my client thing" hook → either a `Component` or a `GameObjectSystem` + `OnClientInitialize`. Components are convenient when the logic naturally lives on a specific GameObject.

- **Dedicated server.** `OnClientInitialize` is **not** called on a dedicated server, by design — it's for client-side concerns. Host machines that are also playing get it right after `OnHostInitialize`.
- **Editor.** None of these methods fire in the editor. The XML doc says explicitly "In game only" for the host callbacks; `OnClientInitialize` follows the same in-game-only logic via the loading flow.
- **Spawning networked objects in `OnClientInitialize`.** If the local client is the lobby host, the scene is also being snapshotted to other clients during start-up. Spawning a *networked* object here will replicate, then your remote clients will spawn it too in their own `OnClientInitialize` — leading to duplicates. Either restrict the spawn to non-host clients, or move it to `OnHostInitialize` and let replication carry it across.
- **Order between subscribers.** Multiple `ISceneStartup` subscribers are dispatched together with no guaranteed ordering between them. If you have ordering dependencies, encode them in a single subscriber rather than across two.

## Related Pages

- [Component Events](index.md)
- [Component Methods Reference](../component-methods.md)
- [GameObject System](../../gameobjectsystem.md)
- [IGameObjectNetworkEvents](igameobjectnetworkevents.md)
