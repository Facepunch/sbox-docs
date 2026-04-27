---
title: "Component Reference"
icon: "🔗"
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.Reference.cs
created: 2026-04-27
updated: 2026-04-27
---

# Component Reference

The `ComponentReference` struct allows you to store a serialized reference to a specific `Component` on a `GameObject`, maintaining the connection even if the object is sent across the network, serialized to disk, or reloaded.

## Quick Working Example

You can define a property of type `ComponentReference` inside your component to link to another component in the scene.

```csharp
using Sandbox;

public class MyTrigger : Component
{
	// Store a reference to a target component
	[Property] public ComponentReference TargetComponentRef { get; set; }

	public void Activate()
	{
		// Attempt to resolve the reference at runtime
		var component = TargetComponentRef.Resolve();

		if ( component != null )
		{
			Log.Info( $"Successfully found {component.GameObject.Name}" );
		}
		else
		{
			Log.Warning( "Failed to resolve the component reference!" );
		}
	}
}
```

### Creating a Reference from Code

If you want to dynamically create and store a reference to an existing component, use `ComponentReference.FromInstance()`.

```csharp
public void StoreTarget( Component target )
{
	TargetComponentRef = ComponentReference.FromInstance( target );
}
```

### Resolving to a Specific Type

If you know the type of component you expect, you can pass the type to `Resolve()`.

```csharp
var player = TargetComponentRef.Resolve( Game.ActiveScene, typeof(PlayerController) ) as PlayerController;
```

## Configuration

The struct itself exposes several readonly properties that identify the target component:

| Property | Description |
|---|---|
| `ComponentId` | The unique `Guid` of the referenced component. |
| `GameObjectId` | The `Guid` of the `GameObject` containing the referenced component. |
| `ComponentTypeName` | The class name (`TypeDescription.ClassName`) of the component, if available. |
| `ReferenceType` | Always `"component"` for a component reference. |

## Troubleshooting

:::warning Resolve() returns null
If `.Resolve()` returns `null`, the target component or its parent `GameObject` may have been destroyed. It can also happen if you're loading a scene where the object no longer exists, or if you're receiving a network update for an object that hasn't spawned on your client yet.
:::

## Related Pages
- [GameObject System](../gameobjectsystem.md)
- [Components Overview](index.md)
