---
title: "Events"
icon: "🛜"
sources:
  - engine/Sandbox.Engine/Scene/Scene/Scene.Event.cs
  - engine/Sandbox.Engine/Scene/Events/ISceneCollisionEvents.cs
  - engine/Sandbox.Engine/Scene/Events/ISceneLoadingEvents.cs
  - engine/Sandbox.Engine/Scene/Events/IScenePhysicsEvents.cs
  - engine/Sandbox.Engine/Scene/Events/ISceneStartup.cs
  - engine/Sandbox.Engine/Scene/Events/IGameObjectNetworkEvents.cs
created: 2026-04-27
updated: 2026-04-27
---

# Events

Events allow components and systems to broadcast and listen to messages locally within a scene using interfaces.

## Quick Working Example

```csharp
// 1. Define the event interface
public interface IPlayerSpawnEvent : ISceneEvent<IPlayerSpawnEvent>
{
    void OnSpawned( GameObject player );
}

// 2. Implement the interface on any Component to listen
public class SpawnSound : Component, IPlayerSpawnEvent
{
    void IPlayerSpawnEvent.OnSpawned( GameObject player )
    {
        Log.Info( $"{player.Name} has spawned!" );
    }
}

// 3. Broadcast the event from anywhere
IPlayerSpawnEvent.Post( x => x.OnSpawned( GameObject ) );
```

### Broadcasting to the Entire Scene

You can broadcast any interface (even built-in ones) to all active components in the scene using `Game.ActiveScene.RunEvent<T>`.

```csharp
// Tint all ModelRenderers in the scene to red
Game.ActiveScene.RunEvent<ModelRenderer>( x => x.Tint = Color.Red );

// Call a custom interface method
Game.ActiveScene.RunEvent<IDamageable>( x => x.OnDamage( damageInfo ) );
```

### Broadcasting to a Specific GameObject

Sometimes you only want to send an event to a specific object and its children, not the whole world.

```csharp
// Only components on the 'enemy' GameObject (or its children) will receive this
enemy.RunEvent<IDamageable>( x => x.OnDamage( damageInfo ) );
```

### Using ISceneEvent&lt;T&gt; (Syntax Sugar)

If you define a custom event, inheriting from `ISceneEvent<T>` gives you access to the static `.Post()` method, which makes broadcasting cleaner.

```csharp
public interface IMyEvent : ISceneEvent<IMyEvent>
{
    void DoSomething();
}

// Broadcasts to the whole scene
IMyEvent.Post( x => x.DoSomething() );
```

### Optional Event Methods

If your interface has many methods but listeners usually only need one, provide empty default implementations in the interface. This prevents listeners from having to implement every single method.

```csharp
public interface IPlayerEvent : ISceneEvent<IPlayerEvent>
{
    void OnJump() { }
    void OnLand( float distance ) { }
    void OnTakeDamage( float damage ) { }
}
```

---

## Built-in Engine Events

The engine provides several built-in event interfaces you can implement to hook into engine systems.

### ISceneCollisionEvents

Listen to **all** collision events that happen anywhere in the scene during a physics step.

```csharp
public sealed class CollisionSoundSystem : GameObjectSystem<CollisionSoundSystem>, ISceneCollisionEvents
{
	public CollisionSoundSystem( Scene scene ) : base( scene ) { }

	void ISceneCollisionEvents.OnCollisionHit( Collision collision )
	{
		if ( collision.Contact.Speed.Length > 100f )
		{
			Log.Info( $"Global Hit: {collision.Self.GameObject.Name} hit {collision.Other.GameObject.Name}!" );
		}
	}
}
```

| Method | Description |
|---|---|
| `OnCollisionStart( Collision collision )` | Called when a collider/rigidbody starts touching another collider anywhere in the scene. |
| `OnCollisionUpdate( Collision collision )` | Called once per physics step for every collider being touched. |
| `OnCollisionStop( CollisionStop collision )` | Called when a collider/rigidbody stops touching another collider. |
| `OnCollisionHit( Collision collision )` | Called when a collider/rigidbody hits another collider, including repeated hits. |

:::danger Getting Too Many Events!
If you attach a component with `ISceneCollisionEvents` to a player, it will fire every time a physics barrel hits the floor on the other side of the map!

If you only want to listen to collisions involving *your specific GameObject*, use **`Component.ICollisionListener`** instead.
:::

### IScenePhysicsEvents

This interface allows you to run logic immediately before or after the physics step, or react to Rigidbody sleeping states. 

```csharp
public class CustomGravity : Component, IScenePhysicsEvents
{
	[RequireComponent] Rigidbody myBody { get; set; }

	void IScenePhysicsEvents.PrePhysicsStep()
	{
		if ( myBody.IsValid() )
			myBody.ApplyForce( Vector3.Up * 100f );
	}
}
```

| Method | Description |
|---|---|
| `PrePhysicsStep()` | Called right before the physics simulation step (typically right after FixedUpdate). |
| `PostPhysicsStep()` | Called immediately after the physics simulation step finishes. |
| `OnOutOfBounds( Rigidbody body )` | Called when a rigidbody falls out of the world bounds. |
| `OnFellAsleep( Rigidbody body )` | Called when a rigidbody stops moving and goes to sleep. |

:::warning FixedUpdate vs PrePhysicsStep
`OnFixedUpdate` runs on all components before `PrePhysicsStep`. Use `PrePhysicsStep` specifically when you need to be the absolute last thing that happens before the physics solver runs, ensuring no other component overrides your forces.
:::

### ISceneLoadingEvents

Allows listening to events related to scene loading, including asynchronous tasks.

```csharp
public class SceneLoaderHooks : Component, ISceneLoadingEvents
{
	Task ISceneLoadingEvents.OnLoad( Scene scene, SceneLoadOptions options )
	{
		Log.Info("Doing custom setup...");
		return Task.CompletedTask;
	}
}
```

| Method | Description |
|---|---|
| `BeforeLoad( Scene scene, SceneLoadOptions options )` | Called before the loading starts. |
| `OnLoad( Scene scene, SceneLoadOptions options )` | Called during loading. The game will wait for your task to finish. |
| `AfterLoad( Scene scene )` | Loading has finished. |

:::danger Unbounded Tasks
When implementing `OnLoad`, the game completely halts the scene transition until the `Task` completes. Ensure any asynchronous operations have proper timeouts or error handling, otherwise the game will hang indefinitely on the loading screen!
:::

### ISceneStartup

The `ISceneStartup` event interface allows `GameObjectSystem`s and components to listen to Scene startup events to initialize logic when a game or server starts.

```csharp
public sealed class MyGameManager : GameObjectSystem, ISceneStartup
{
	public MyGameManager( Scene scene ) : base( scene ) { }

	void ISceneStartup.OnHostInitialize()
	{
		Log.Info("Host has initialized the scene!");
	}
}
```

| Method | When it is called | Typical Use Case |
|---|---|---|
| `OnHostPreInitialize( SceneFile scene )` | Before the scene is loaded on the host (scene is empty). | Setup for `GameObjectSystem` since Components don't exist yet. |
| `OnHostInitialize()` | After the scene is loaded on the host. | Spawning host-controlled objects like game managers or common cameras. |
| `OnClientInitialize()` | After the scene is loaded on client and host. | Spawning client-side visual effects or UI. |

:::warning Networked Objects in Client Init
If you spawn objects in `OnClientInitialize`, ensure they are not networked. If the client is also the lobby host, the scene will be snapshotted and sent to other clients, causing duplicates!
:::

### IGameObjectNetworkEvents

Allows a GameObject's Components to react when its network ownership or control state changes.

```csharp
public class Vehicle : Component, IGameObjectNetworkEvents
{
	void IGameObjectNetworkEvents.StartControl()
	{
		Log.Info("We are now driving!");
	}

	void IGameObjectNetworkEvents.StopControl()
	{
		Log.Info("Someone else took the wheel, or we exited");
	}
}
```

| Method | Description |
|---|---|
| `NetworkOwnerChanged( Connection newOwner, Connection previousOwner )` | Called when the owner of the network GameObject changes. |
| `StartControl()` | Called when the local client becomes the controller of this object. |
| `StopControl()` | Called when this object becomes a proxy (controlled by someone else). |

:::success Targeted Event
This event is only broadcasted to the single GameObject that is changing ownership, not the entire scene!
:::

---

## Troubleshooting

:::warning Not a Network Event
`Scene.RunEvent` and `ISceneEvent.Post` **do not** send messages over the network to other players. They only execute locally on the machine that calls them. For networking, look into RPC Messages.
:::

:::danger Listeners Must Be Active
Events are only dispatched to Components and GameObjects that are currently **enabled**. If a component is disabled, it will not receive the event.
:::

## Related Pages
* [Components Overview](../index.md)
* [GameObject](../../gameobject.md)
