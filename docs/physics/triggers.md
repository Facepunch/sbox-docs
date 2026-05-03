---
title: "Triggers"
icon: "🚪"
created: 2026-05-03
updated: 2026-05-03
---

# Triggers

A trigger is a collider that detects when other physics objects enter or exit it, **without blocking their movement**. They're useful for things like pickup zones, death planes, checkpoints, and area detection.


# Setting Up a Trigger

Any collider can become a trigger. In the inspector, tick the **Is Trigger** checkbox on a `BoxCollider`, `SphereCollider`, `CapsuleCollider`, or any other collider component.

To do it in code:

```csharp
var box = AddComponent<BoxCollider>();
box.Scale = new Vector3( 200, 200, 100 );
box.IsTrigger = true;
```


# Reacting to Trigger Events

Add `Component.ITriggerListener` to any component on the same GameObject as the trigger collider (or a child). Implement `OnTriggerEnter` and `OnTriggerExit`:

```csharp
public sealed class PickupZone : Component, Component.ITriggerListener
{
	public void OnTriggerEnter( Collider other )
	{
		Log.Info( $"{other.GameObject.Name} entered the pickup zone" );
	}

	public void OnTriggerExit( Collider other )
	{
		Log.Info( $"{other.GameObject.Name} left the pickup zone" );
	}
}
```

You can also use the overloads that give you both colliders if you need to know which trigger was hit (useful when one GameObject has multiple triggers):

```csharp
public void OnTriggerEnter( Collider self, Collider other )
{
	Log.Info( $"{other.GameObject.Name} entered {self.GameObject.Name}" );
}
```

And there are `GameObject` variants if you only care about the entering object, not its specific collider:

```csharp
public void OnTriggerEnter( GameObject other ) { }
public void OnTriggerExit( GameObject other ) { }
```

:::info
The default player controller setup uses two child colliders for the player: a capsule for the body and a box for the feet. When entering a trigger, you might get two events (one for each collider), and neither of them will be the actual player GameObject. Use `FindMode.InAncestors` to check for the player component in the parent GameObject.
:::

# Checking What's Inside (Polling)

If you'd rather poll than use events, `Collider.Touching` gives you all colliders currently overlapping this trigger:

```csharp
[Property] public BoxCollider Box { get; set; }

protected override void OnUpdate()
{
    foreach ( var collider in Box.Touching )
    {
        Log.Info( $"{collider.GameObject.Name} is inside the box" );
    }
}
```


:::info
The trigger fires `OnTriggerExit` automatically when a collider inside the trigger is disabled or destroyed, so you don't need to manually clean up the list for disabled objects.
:::
