---
title: "Networking"
icon: "🌐"
created: 2026-04-11
updated: 2026-04-11
---

# Networking

Maps and networking work together automatically. Most of the map's content does not need to be networked because it is loaded identically on every client from the same map file.


# What Is Networked

Physics props (`prop_physics`) are the main exception. When the map loads, each physics prop is spawned as a fully networked object.

```
prop.GameObject.Network.SetOrphanedMode( NetworkOrphaned.ClearOwner );
prop.GameObject.Network.SetOwnerTransfer( OwnerTransfer.Takeover );
prop.GameObject.NetworkSpawn();
```

This means any client can interact with physics props - pushing, shooting, or destroying them - and the changes replicate to everyone.


# What Is Not Networked

Everything else in the map is loaded locally and does not go through the network system:

- World geometry and collision
- Lights and light probes
- Fog volumes
- Skyboxes
- Soundscapes
- Static props and `func_brush` entities
- Cubemap probes

These are all deterministic - they produce the same result on every client because they come from the same map file.


# Client Joining

When a client joins a game, the `MapInstance` is part of the scene snapshot. The map loads on the client and all its entities are created locally.

The `MapInstance` is careful not to duplicate objects that already exist from the network. If a child GameObject was already received via the network snapshot, the map loader will skip creating it again. This prevents doubled props or conflicting state.


# Scene Map GameObjects

Maps can also contain [GameObjects](/scene/gameobject.md) embedded in a scene file (`world.scene`). These are deserialized and added as children of the `MapInstance`. Any of these GameObjects that have `NetworkMode.Object` set will be network spawned automatically by the host.

On the client side, GameObjects that arrive via the network are not re-created from the map file - the networked version takes precedence.
