---
title: "Component Interfaces"
icon: "🧸"
created: 2024-03-10
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Markers/DontExecuteOnServer.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/ITintable.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/ExecuteInEditor.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/ICollisionListener.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/ITriggerListener.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/IDamageable.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/INetworkListener.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/INetworkSpawn.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/IColorProvider.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/IHasBounds.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/IMaterialSetter.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/ISceneStage.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/IPressable.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/INetworkVisible.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/INetworkSnapshot.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/ITemporaryEffect.cs
---

# Component Interfaces

The `Component` class provides several built-in interfaces you can implement to listen to specific engine events or enable special behaviors. Think of these as "opt-in plugins" for your components.

### 1. Implementing an Interface
You implement an interface like any standard C# interface.
```csharp
public sealed class MyDestructibleBox : Component, Component.IDamageable
{
	public void OnDamage( in DamageInfo damage )
	{
		Log.Info("Box was damaged!");
	}
}
```

### 2. Broadcasting via Game.ActiveScene
You can call an interface method on all active instances using `RunEvent`.
```csharp
Game.ActiveScene.RunEvent<Component.IDamageable>( x => x.OnDamage( new DamageInfo() ) );
```

### 3. Fetching interface implementers
You can fetch all components implementing an interface dynamically.
```csharp
var damageables = Scene.GetAll<Component.IDamageable>();
foreach ( var damageable in damageables )
{
    // Do something
}
```

## Core Capabilities

Modify when or how your component executes within the engine's lifecycle.

| Interface | Description | Typical Use Case |
|---|---|---|
| **`ExecuteInEditor`** | A marker interface that tells the engine to run update loops (`OnUpdate`, etc.) while editing the scene. | Drawing custom gizmos, or components that automatically snap to the grid in the editor. |
| **`DontExecuteOnServer`** | A marker interface that prevents execution on dedicated servers (client-only). | Visual effects, UI elements, or client-side only audio spawners. |
| **`ISceneStage`** | Hook into the start (`Start()`) or end (`End()`) of the scene tick loop. | Low-level systems that need to strictly run before or after all other components update. |

### Example: ExecuteInEditor
```csharp
public sealed class ExecuteInEditorSample : Component, Component.ExecuteInEditor
{
	protected override void OnEnabled()
	{
		base.OnEnabled();
		// Separate editor logic from gameplay logic
		if ( Game.IsEditor )
		{
			Log.Info( "OnEnabled is executing while editing the scene!" );
		}
	}
}
```

---

## Physics & Interactions

Hook into the physics system to respond to physical events or player interactions. See the [Collisions and Triggers](../../../systems/physics/collisions-and-triggers.md) guide for more details.

| Interface | Description | Typical Use Case |
|---|---|---|
| **`ICollisionListener`** | Callbacks (`OnCollisionStart`, `OnCollisionUpdate`, `OnCollisionStop`) when colliders crash. | Dealing damage on impact, playing collision sounds, or bouncing behavior. |
| **`ITriggerListener`** | Callbacks (`OnTriggerEnter`, `OnTriggerExit`) when objects enter trigger volumes. | Spikes, jump pads, or invisible zone triggers that start a cutscene. |
| **[`IDamageable`](idamageable.md)** | One method: `OnDamage(in DamageInfo damage)`. The standard contract every damage system targets. | Health components for players, enemies, destructible props. *(Dedicated page covers `DamageInfo`, hitbox multipliers, multiplayer authority.)* |
| **[`IPressable`](ipressable.md)** | Full interaction lifecycle (`Hover`/`Look`/`Blur`/`Press`/`Pressing`/`Release`) plus `GetTooltip`. | Buttons, doors, pickup items, sittable seats. *(Dedicated page covers the lifecycle order and Tooltip semantics.)* |

:::warning Colliders Required
Interfaces like `ICollisionListener`, `ITriggerListener`, and `IPressable` generally require a `Collider` component on the same `GameObject` or its children to function. Triggers must have the **Is Trigger** property checked.
:::

### Example: IPressable
```csharp
public sealed class InteractiveButton : Component, Component.IPressable
{
	public bool Press( IPressable.Event e )
	{
		Log.Info( $"{e.Source} pressed me!" );
		return true; // Return true to indicate successful press
	}

	public IPressable.Tooltip? GetTooltip( IPressable.Event e )
	{
		return new IPressable.Tooltip( "Press Me", "touch_app", "Click to interact" );
	}
}
```

---

## Networking

Respond to multiplayer state changes. *(See [Network Events](../../../systems/networking-multiplayer/network-events.md) for more details).*

| Interface | Description |
|---|---|
| **`INetworkListener`** | React to global network lifecycle events like `OnConnected`, `OnDisconnected`, and `OnActive`. |
| **`INetworkSpawn`** | Provides `OnNetworkSpawn(Connection)` to run logic right after this object is instantiated over the network. |
| **`INetworkVisible`** | Determine object visibility per-connection via `IsVisibleToConnection(Connection, in BBox)`. |
| **`INetworkSnapshot`** | Manually serialize complex data into/out of network snapshots via `WriteSnapshot` and `ReadSnapshot`. |

---

## Rendering & Visuals

Standardize how visual components report or change their state.

| Interface | Description | Typical Use Case |
|---|---|---|
| **`ITintable`** | Provides a unified `Color` property (`Color { get; set; }`). | Allowing other scripts to easily flash an object red when it gets hurt, without knowing its exact renderer type. |
| **`IMaterialSetter`** | Standardize setting and getting materials (`SetMaterial`, `GetMaterial`). | Components that apply custom skins or damage decals to an object. |
| **`IColorProvider`** | Provides a custom `ComponentColor` (`Color { get; }`) for engine Editor UI elements. | Changing the color of your custom component's icon or node in the ActionGraph or Inspector. |
| **`IHasBounds`** | Exposes the bounding box of a component in local space via `LocalBounds`. | Custom culling, or querying the physical volume of a procedurally generated mesh. |
| **[`ITemporaryEffect`](itemporaryeffect.md)** | `IsActive` plus optional `DisableLooping()` — lets `TemporaryEffect` know when it's safe to delete the host GameObject. | Particle spawners, beam effects, decals, sound emitters that need to wind down gracefully. *(Dedicated page covers the polling model and the `DisableLoopingEffects` static helper.)* |

### Example: ITintable

```csharp
// By implementing ITintable, other scripts can universally change our color.
public sealed class MyInteractiveBox : Component, Component.ITintable, Component.ICollisionListener
{
	[Property] public Color Color { get; set; } = Color.White;

	public void OnCollisionStart( Collision collision )
	{
		// Flash red when we hit something
		Color = Color.Red;
	}
}
```

## Troubleshooting

:::danger Desynchronization with DontExecuteOnServer
Do not put any game logic (like dealing damage, modifying networked variables, or moving physical objects) inside a component with `DontExecuteOnServer`. Doing so will cause the clients to get out of sync with the authoritative server!
:::

:::warning Not finding my interface
When using `Scene.GetAll<IMyInterface>()`, make sure the components are actually enabled and that the GameObjects they are on are also enabled. The scene only tracks active components.
:::

## Related Pages
* [Component Lifecycle](../component-lifecycle.md)
* [Collisions and Triggers](../../../systems/physics/collisions-and-triggers.md)
* [Networking & Multiplayer](../../../systems/networking-multiplayer/index.md)
* [GameObject](../../gameobject.md)
* [Components](../index.md)
