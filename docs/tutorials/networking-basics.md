---
title: "Build a Networked Lobby"
icon: "🧑‍🤝‍🧑"
sources:
  - engine/Sandbox.Engine/Systems/Networking/Networking.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/INetworkListener.cs
  - engine/Sandbox.Engine/Scene/Networking/Sync.cs
  - engine/Sandbox.Engine/Scene/Networking/Rpc.Attributes.cs
  - engine/Sandbox.Engine/Scene/Networking/NetworkObject.cs
updated: 2026-04-25
created: 2026-04-27
---

# Build a Networked Lobby

> **New to networking?** Read **[Networking & Multiplayer](../systems/networking-multiplayer/index.md)** first — that's the overview. This page is the **tutorial** for building a networked lobby end-to-end.

A complete networked game loop in one file: a host opens a lobby, other clients join, each is given their own player, and the player has a synced health value plus an RPC for taking damage. By the end you'll have a two-instance local test running.

## What you'll build

- `NetworkManager` — creates a Steam lobby on game start and spawns a player for each connection.
- A networked Player prefab with `NetworkMode.Object` set, a `[Sync]`'d health value, and a `[Rpc.Broadcast]` damage method.
- A simple "click to take damage" interaction to test the RPC end-to-end.

## Prerequisites

- [Your First Component](your-first-component.md) and [Your First Game](your-first-game.md) recommended.
- A working player setup. Re-use the `Player` prefab from the third-person controller tutorial, or build a minimal one (Empty GameObject + `ModelRenderer` + `PlayerController` + child `Camera` with `CameraComponent`).

> The engine's networking is built on Steam's relay infrastructure for production, but in-editor multi-instance testing works without involving Steam's matchmaker. You don't need a real friend to test this tutorial.

---

## Step 1: The NetworkManager

We need one component that runs on every instance and knows whether to start a lobby or join one. Create `NetworkManager.cs`:

```csharp
using Sandbox;
using Sandbox.Network;
using System.Threading.Tasks;

public sealed class NetworkManager : Component, Component.INetworkListener
{
    [Property] public GameObject PlayerPrefab { get; set; }
    [Property] public List<GameObject> SpawnPoints { get; set; } = new();

    protected override async Task OnLoad()
    {
        if ( Scene.IsEditor )
            return;

        // If we're not already in a session (i.e., we weren't joined by an invite),
        // we are the first to enter - start a lobby for others to join.
        if ( !Networking.IsActive )
        {
            await Networking.CreateLobby( new LobbyConfig()
            {
                MaxPlayers = 8,
                Privacy = LobbyPrivacy.Public,
                Name = "My First Lobby",
            } );
        }
    }
}
```

A few things to know:

- **`Component.INetworkListener`** is the interface for receiving connection events. Implementing it is what registers your component with the lobby system. Methods on the interface (`OnActive`, `OnConnected`, `OnDisconnected`, `OnBecameHost`) only fire on the host.
- **`OnLoad`** runs *before* the scene starts ticking — that's why we use it instead of `OnStart` for the lobby creation. It returns a `Task`, so `await` works naturally.
- **`Scene.IsEditor`** prevents the lobby from being created when you're just placing things in the editor view, only at actual game time.
- **`!Networking.IsActive`** means "no networking session is currently set up". When a friend invites you to their lobby, `Networking.IsActive` will already be true by the time `OnLoad` fires, so you skip lobby creation.

> **If `OnLoad` doesn't seem to be called**, you put your code in `OnStart` by mistake — `OnLoad` is its own override on `Component` (defined in `Component.Loading.cs`) and only runs when the scene is loading.

> **If `Networking.CreateLobby` errors out**, Steam isn't initialized. The Editor brings Steam up automatically; if you're running outside the Editor (a packaged build), you need a `steam_appid.txt` next to the executable.

### Checkpoint

Place a `NetworkManager` in your scene (Empty GameObject with the component attached). Press Play. Look at the console — you should see the lobby system print initialization messages, ending with something like `Lobby Created: My First Lobby`.

---

## Step 2: Spawning players on connect

`OnActive` is called on the host every time a player finishes connecting (including the host themselves). Add this method:

```csharp
public void OnActive( Connection connection )
{
    Log.Info( $"Player {connection.DisplayName} connected!" );

    if ( PlayerPrefab is null )
    {
        Log.Warning( "NetworkManager has no PlayerPrefab assigned!" );
        return;
    }

    // Pick a spawn point.
    var startTransform = SpawnPoints.Count > 0
        ? SpawnPoints[ Game.Random.Int( 0, SpawnPoints.Count - 1 ) ].WorldTransform
        : GameObject.WorldTransform;

    // Clone the prefab. This only runs on the host.
    var playerGo = PlayerPrefab.Clone( startTransform );
    playerGo.Name = $"Player - {connection.DisplayName}";

    // Network spawn it. This sends the spawn to every connected client.
    // The connection passed becomes the *owner* - they get IsProxy = false.
    playerGo.NetworkSpawn( connection );
}
```

Key API:

- **`PlayerPrefab.Clone( transform )`** — creates a fresh GameObject from the prefab. At this point it's a normal local object, identical to dragging the prefab into the scene.
- **`NetworkSpawn( connection )`** — registers the GameObject with the network system. Every other connected client receives a snapshot of it within the next tick. The `connection` argument becomes the new GameObject's *owner*; on the owner's machine, `IsProxy` returns false. Everyone else sees `IsProxy = true`.
- The whole `OnActive` body **only runs on the host**, because that's the only place `INetworkListener.OnActive` fires.

> **If players don't spawn on the host, only on clients (or vice versa)**, double-check who you're calling `NetworkSpawn` from. The host clones-and-spawns; the clients get the prefab automatically through replication. Calling `NetworkSpawn` from a client fails silently.

---

## Step 3: Networking the player prefab

For replication to work, the prefab itself has to be marked as a network object.

1. Open your player prefab.
2. Select the **root GameObject** of the prefab.
3. In the Inspector, find `NetworkMode`.
4. Set it to `Object`. (The other values: `Snapshot` is the default — replicated only when the host explicitly broadcasts a scene snapshot. `Never` means the object is host-only.)
5. Save the prefab.

> **If `NetworkMode` isn't visible**, you're looking at a child of the prefab, not the root. Click the topmost row in the prefab's tree.

> **If you set `NetworkMode = Object` and then `NetworkSpawn` immediately throws "already networked"**, that's because `NetworkMode = Object` on a *placed* prefab in a scene auto-spawns it on session start. For a prefab you intend to clone-and-spawn (like in this tutorial), it's fine — the per-instance `NetworkSpawn` is what registers the *clone*.

---

## Step 4: Hook everything together in the scene

1. In the scene, drop a few empty GameObjects to act as spawn points. Name them `SpawnPoint_1`, `SpawnPoint_2`, etc., and place them around the floor.
2. Select `NetworkManager`. In the Inspector, drag your player prefab into `Player Prefab`.
3. Drag each spawn-point GameObject into the `Spawn Points` list.

### Checkpoint

In the Editor, find the play-mode multi-instance setting (in the **Play** dropdown next to the play button — there's an option for the number of instances to launch). Set it to 2.

Press Play. Two windows open. The first is the host; it creates the lobby. The second auto-joins. Each window's console should print `Player <name> connected!` for both players, and you should see two character GameObjects in each window's scene.

> **If only one window connects**, the second instance is failing the matchmaker handshake. Check the second window's console for messages like "could not find lobby". If you see those, the lobby visibility is wrong — make sure `LobbyPrivacy.Public` is set in `CreateLobby`.

---

## Step 5: Sync a property — health

Up to now players just *exist* on every machine. Let's add a value that changes during play, and watch it propagate.

On your player prefab's root, add a new component `PlayerStats.cs`:

```csharp
using Sandbox;

public sealed class PlayerStats : Component
{
    /// <summary>
    /// Synced from the owner of this object. When the owner sets MaxHealth or Current,
    /// every other client sees the change automatically.
    /// </summary>
    [Sync] public float MaxHealth { get; set; } = 100f;

    [Sync, Change( nameof( OnCurrentChanged ) )]
    public float Current { get; set; } = 100f;

    protected override void OnUpdate()
    {
        // We only want the owner to mutate Current. Proxies (other people's view of this object) skip.
        if ( IsProxy ) return;

        // Trickle health down for demonstration.
        Current = MathF.Max( 0f, Current - Time.Delta * 5f );
    }

    void OnCurrentChanged( float oldValue, float newValue )
    {
        Log.Info( $"{GameObject.Name}.Current: {oldValue:F1} -> {newValue:F1} (IsProxy={IsProxy})" );
    }
}
```

> **Pause and predict:** Inside `OnUpdate` we have `if ( IsProxy ) return;`. Imagine we deleted that line. With two clients connected and each having their own `PlayerStats` for both players, what *exactly* happens to the `Current` value? Walk through it tick by tick.
>
> _Continue reading once you've made a guess._

The new ingredients:

- **`[Sync]`** — marks a property for automatic synchronization. The owner writes; every other client reads. The framework batches changes and replicates them at the next snapshot.
- **`IsProxy`** — true on every machine *except* the owner. The host sees `IsProxy = false` for the host's own player and `IsProxy = true` for everyone else's. Use this guard to make sure only the authoritative side runs simulation logic.
- **`[Change(nameof(OnCurrentChanged))]`** — runs the named method whenever the property's value changes. The signature is always `(T oldValue, T newValue)`. If you omit the name argument (`[Change]`), it looks for `On<PropertyName>Changed`.

> **If `[Sync]` doesn't compile**, you forgot `using Sandbox;`. The attribute lives in the `Sandbox` namespace.

> **If your sync seems to be running both ways** (everyone modifying everyone's health), you forgot the `IsProxy` guard. Without it, every machine ticks `Current` for every player, and they fight each other.

### Checkpoint

Press Play with two instances. In each window, watch the console — you should see `OnCurrentChanged` fire on both. Crucially, the *owner's* window says `IsProxy=False` and the *other* window says `IsProxy=True`, both for the same player.

The values stay in sync within a fraction of a second.

---

## Step 6: Send an RPC

Sometimes you don't want to sync continuous state — you want to *call a method* on every machine. That's an RPC.

Add to `PlayerStats`:

```csharp
[Rpc.Broadcast]
public void TakeDamage( float amount )
{
    Log.Info( $"TakeDamage({amount}) ran on {(IsProxy ? "proxy" : "owner")}" );

    // Only the owner is authoritative for the health number, but the visual effect
    // can run on every client.
    if ( !IsProxy )
        Current = MathF.Max( 0f, Current - amount );

    // Imagine a particle effect spawning here on every machine.
}

protected override void OnUpdate()
{
    if ( !IsProxy && Input.Pressed( "attack1" ) )
    {
        TakeDamage( 10f );
    }
}
```

> **Pause and predict:** When a client calls `TakeDamage(10f)` on their own player, the method runs locally *and* on every other connected machine. But inside the method we wrote `if ( !IsProxy ) Current = ...`. Why guard the assignment when we *want* the damage to apply everywhere? What would happen if every machine wrote to `Current` directly?
>
> _Continue reading once you've made a guess._

`[Rpc.Broadcast]` (defined in `Sandbox.Rpc`) means: when this method is invoked, the call is replicated to every connected client (including the caller) and runs there. The `Rpc` namespace also offers:

- **`[Rpc.Owner]`** — runs only on the owner of the object the method belongs to.
- **`[Rpc.Host]`** — runs only on the session host.

> **If your RPC fires twice on the caller**, that's the broadcast model: one local call, one received-from-network call. If you don't want both, gate one side with `Rpc.Caller != Connection.Local` (advanced usage).

> **If your RPC doesn't fire on the other client**, the GameObject this method belongs to isn't actually networked. Check `NetworkMode = Object` on the prefab root, and confirm `NetworkSpawn` was called.

### Checkpoint

Run two instances. In the host window, click left mouse (the default `attack1` binding). Both windows log `TakeDamage(10) ran on ...`. The `Current` health value drops by 10 on the owner only — but because `Current` is `[Sync]`, the new value replicates to the other window within a frame, so they end up matching.

---

## Step 7: Test the loop

You now have:

1. A **host** that opens a lobby on game start.
2. **Spawned players**, one per connection, each with the host as the source of truth on their existence.
3. **Synced state** (`Current` health) where the owner's value broadcasts to everyone.
4. **An RPC** triggered by player input, executing the damage application authoritatively while replicating the visual side to every machine.

That's the canonical multiplayer pattern: state-replicating fields plus method-replicating RPCs.

---

:::danger Player spawned locally but other clients can't see them
The prefab root needs `NetworkMode = Object`, and `NetworkSpawn(connection)` must run **on the host**. If you accidentally call `NetworkSpawn` from a non-host context, it silently does nothing.
:::

:::warning All clients run the simulation, not just the owner
You didn't guard with `if ( IsProxy ) return;` in your `OnUpdate`. Every machine has its own copy of the GameObject and ticks every component. Without the guard you get a multi-way write conflict.
:::

:::warning RPC doesn't reach the other client
`[Rpc.Broadcast]` requires the GameObject to be networked. If `NetworkMode != Object`, the call runs locally but isn't transmitted. Also: a method must be on a `Component` for the RPC code generator to wrap it.
:::

:::tip "I want to test multiplayer alone"
Use the editor's multi-instance play mode (set the instance count to 2 in the Play settings). The first instance becomes the host; the second auto-joins via the in-process matchmaker.
:::

## Try extending it

- **Despawn on disconnect.** Add `OnDisconnected( Connection c )` to the `NetworkManager` and `Scene.GetAllComponents<PlayerStats>()` to find the matching player by ownership, then `GameObject.Destroy()` it.
- **Spectator support.** When the lobby is full (`Connection.All.Count >= MaxPlayers`), don't spawn a player; instead attach the camera to an existing player.
- **Health bar.** Read `PlayerStats.Current` from a Razor panel ([Build a UI](build-a-ui.md) covers the syntax). Render it above the player's head with a `WorldPanel`.
- **Server-authoritative damage.** Convert `TakeDamage` from `[Rpc.Broadcast]` to `[Rpc.Host]` so only the host's call has authority over `Current`. The owner notifies the host with an RPC; the host applies the change and the `[Sync]` propagates.

## Try it yourself

Now add a **second RPC: `Heal`**. The brief: pressing `attack2` heals the local player by 25 (capped at `MaxHealth`), and the heal should be visible on every other connected machine. This uses everything you already wrote — no new APIs, just a second RPC and a second input check.

Your task:
- Add a `Heal( float amount )` method on `PlayerStats` decorated with `[Rpc.Broadcast]`.
- In `OnUpdate`, when the local owner presses `attack2`, call `Heal( 25f )`.
- Apply the heal authoritatively (only on the owner) and clamp to `MaxHealth`.

Predict before you build: when the owner presses `attack2`, will the new value of `Current` reach the other clients via the RPC, the `[Sync]`, or both? And does the answer change if you forget to guard with `IsProxy`?

Try it before reading on. Get stuck? Here's a hint:

<details>
<summary>Hint 1</summary>

The structure mirrors `TakeDamage`:

```csharp
[Rpc.Broadcast]
public void Heal( float amount )
{
    if ( !IsProxy )
        Current = MathF.Min( MaxHealth, Current + amount );
}
```

And the input check goes next to your existing `attack1` check:

```csharp
if ( !IsProxy && Input.Pressed( "attack2" ) )
    Heal( 25f );
```

</details>

<details>
<summary>Hint 2 (the answer)</summary>

```csharp
[Rpc.Broadcast]
public void Heal( float amount )
{
    Log.Info( $"Heal({amount}) ran on {(IsProxy ? "proxy" : "owner")}" );

    if ( !IsProxy )
        Current = MathF.Min( MaxHealth, Current + amount );
}

protected override void OnUpdate()
{
    if ( IsProxy ) return;

    Current = MathF.Max( 0f, Current - Time.Delta * 5f );

    if ( Input.Pressed( "attack1" ) )
        TakeDamage( 10f );

    if ( Input.Pressed( "attack2" ) )
        Heal( 25f );
}
```

The answer to the prediction: the new `Current` value reaches other clients **only via `[Sync]`**, not via the RPC. The RPC's `Heal` call runs on every machine, but the `if ( !IsProxy )` guard means only the owner machine writes `Current`. The owner's write is then replicated by `[Sync]` on the next snapshot.

If you removed the `IsProxy` guard inside `Heal`, every machine would write to its own copy of `Current`. The owner's value would still be authoritative because `[Sync]` overwrites the proxy values from the owner — but for a brief window (between the local write and the next sync tick) the proxies' values would be wrong. With repeated heals you'd see flicker on remote clients. Same hazard as the damage example.

</details>

## Next steps

- The full networking model is described in [Networked Objects](../systems/networking-multiplayer/networked-objects.md) and [Ownership](../systems/networking-multiplayer/ownership.md).
- For a ready-made lobby helper that wraps the boilerplate above, see [Network Helper](../systems/networking-multiplayer/network-helper.md).
- The [Networking & Multiplayer](../systems/networking-multiplayer/index.md) section covers RPCs, sync flags, and connection lifecycle in depth.
