---
title: "Spring Joint"
icon: "🌊"
sources:
  - engine/Sandbox.Engine/Scene/Components/Joint/SpringJoint.cs
updated: 2026-04-25
created: 2026-04-27
---

# Spring Joint

> **New to physics?** Physics overview is at **[Physics](../../../systems/physics/index.md)**. This page is the reference for the `SpringJoint` component.

The `SpringJoint` component connects two physics objects together and tries to keep them a set distance apart, bouncing back like a spring.

## Quick Working Example

```csharp
using Sandbox;

public class BungeeRope : Component
{
	[Property] public GameObject BungeeBody { get; set; }
	[Property] public GameObject AnchorPoint { get; set; }

	protected override void OnStart()
	{
		// Add a spring joint to the bungee body
		var spring = BungeeBody.Components.GetOrCreate<SpringJoint>();
		
		// Connect it to the anchor
		spring.Body = AnchorPoint;
		
		// Configure the spring to only pull, never push
		spring.ForceMode = SpringJoint.SpringForceMode.Pull;
		spring.Frequency = 2.0f; // Stiffness
		spring.Damping = 0.5f; // Bounciness
		spring.MaxLength = 200f; // Rope length before it pulls
		spring.MinLength = 0f;
	}
}
```

### Adjusting Force Mode

You can configure a Spring Joint to act like a rope instead of a solid spring using `ForceMode`.

```csharp
var spring = Components.Get<SpringJoint>();

// A rope can pull objects together if they go too far, but won't push them apart if they get close.
spring.ForceMode = SpringJoint.SpringForceMode.Pull;

// A bumper can push objects away if they get too close, but won't pull them if they move apart.
spring.ForceMode = SpringJoint.SpringForceMode.Push;

// A traditional spring does both.
spring.ForceMode = SpringJoint.SpringForceMode.Both;
```

## Configuration

| Property | Description |
|:---|:---|
| `Frequency` | The stiffness of the spring. Higher values make the spring snap back faster. |
| `Damping` | The damping ratio (usually 0 to 1). 0 makes it bounce forever, 1 makes it settle quickly. |
| `MinLength` | Minimum length it should be allowed to go before strictly stopping compression. |
| `MaxLength` | Maximum length it should be allowed to go before strictly stopping stretching. |
| `RestLength` | The ideal length the spring tries to maintain. |
| `ForceMode` | Determines if the spring pushes (`Push`), pulls (`Pull`), or both (`Both`). |

## Troubleshooting

:::danger "Objects are vibrating or flying away!"
If your objects are violently vibrating or flying away, your `Frequency` (stiffness) is likely too high compared to the `Mass` of your objects. Try lowering the Frequency or increasing the Damping to settle the simulation.
:::

## Related Pages
- [Joints Overview](../../../systems/physics/joints.md)
