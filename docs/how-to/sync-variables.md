---
title: "Sync Variables"
icon: "🔄"
sources:
  - engine/Sandbox.Engine/Scene/Networking/Sync.cs
updated: 2026-04-25
created: 2026-04-27
---

# Sync Variables

> **New to networking?** Read **[Networking & Multiplayer](../systems/networking-multiplayer/index.md)** first — that's the overview. This page is the **recipe** for one specific task: replicating a property's value.

Recipe: share a value across the network so every player sees the same thing.

For the full reference (every `SyncFlags` option, `[Change]` semantics, edge cases, `Query` mode for backing-field bypass) see **[Sync Properties](../systems/networking-multiplayer/sync-properties.md)**. This page is just the minimum you need to copy-paste.

## The Recipe

Add `[Sync]` to a property. Only mutate it from the owner of the GameObject:

```csharp
using Sandbox;

public sealed class ScoreManager : Component
{
    [Sync] public int GlobalScore { get; set; } = 0;

    public void AddScore( int amount )
    {
        // Only the owner (or the host on unowned objects) may write
        if ( IsProxy ) return;
        GlobalScore += amount;
    }
}
```

That's it. The first time the owner changes `GlobalScore`, every connected client sees the new value.

## React to a Change

```csharp
[Sync, Change( nameof(OnScoreChanged) )] public int GlobalScore { get; set; } = 0;

private void OnScoreChanged( int oldValue, int newValue )
{
    Log.Info( $"Score changed from {oldValue} to {newValue}" );
}
```

The handler signature must be `void Method( T oldValue, T newValue )`.

## Sync a Collection

`List<T>` and `Dictionary<K,V>` won't sync — the network has no way to see inserts/removes inside them. Use `NetList<T>` / `NetDictionary<K,V>`:

```csharp
[Sync] public NetList<string> Inventory { get; set; } = new();
```

## "It's not syncing!"

Almost always one of:

- You're writing from a client that doesn't own the object (`IsProxy == true`). Guard with `if ( IsProxy ) return;`.
- You're mutating a private backing field directly instead of going through the setter — see [Sync Properties → Query mode](../systems/networking-multiplayer/sync-properties.md#query-mode-for-complex-getterssetters).
- The GameObject isn't actually networked — call `NetworkSpawn()` once, or set `NetworkMode = Object` in the Inspector. See [Networked Objects](../systems/networking-multiplayer/networked-objects.md).

## Related Pages

- [Sync Properties](../systems/networking-multiplayer/sync-properties.md) — full reference
- [Ownership](../systems/networking-multiplayer/ownership.md) — who is allowed to write
- [Networked Objects](../systems/networking-multiplayer/networked-objects.md) — making a GameObject network-aware
- [RPC Messages](../systems/networking-multiplayer/rpc-messages.md) — when you want to send an event instead of syncing state
