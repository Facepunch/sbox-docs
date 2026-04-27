---
title: "Joints"
icon: "🔗"
sources:
  - engine/Sandbox.Engine/Scene/Components/Joint/Joint.cs
  - engine/Sandbox.Engine/Scene/Components/Joint/HingeJoint.cs
  - engine/Sandbox.Engine/Scene/Components/Joint/SpringJoint.cs
updated: 2026-04-25
created: 2026-04-27
---

# Joints

> **New to physics?** Physics overview is at **[Physics](index.md)**. This page is the reference for joint components.

Joints connect two physics objects together, restricting their movement relative to each other to simulate mechanics like doors, wheels, or chains.

## Quick Working Example

```csharp
public class DoorController : Component
{
	[Property] public GameObject DoorBody { get; set; }
	[Property] public GameObject HingeAnchor { get; set; }

	protected override void OnStart()
	{
		// Add a hinge joint to the door
		var hinge = DoorBody.Components.GetOrCreate<HingeJoint>();
		
		// Connect it to the anchor
		hinge.Body = HingeAnchor;
		
		// Configure limits
		hinge.MinAngle = 0f;
		hinge.MaxAngle = 90f;
	}
}
```

### Connecting to the World
If you don't assign a target `Body` to a Joint, it will connect to the world reference body. This means the object is anchored to its starting position in the scene.

### Breaking Joints
You can configure joints to break when too much force is applied.

```csharp
var joint = Components.Get<HingeJoint>();

// The joint will break if the force exceeds this amount
joint.BreakForce = 5000f;

// Listen for when it breaks
joint.OnBreak += () => {
    Log.Info("The door fell off its hinges!");
};
```

## Joint Types

| Type | Description |
|:---|:---|
| `FixedJoint` | Glues two objects together so they cannot move relative to each other. |
| `HingeJoint` | Constrains objects to rotate around a single axis, like a door or a wheel. |
| [`SpringJoint`](../../scene/components/reference/springjoint.md) | Tries to keep an object a set distance away from another object, bouncing back like a spring. |
| `SliderJoint` | Allows movement along a single line, like a piston. |
| `BallJoint` | Allows rotation in all directions around a single point, like a shoulder joint. |

## Configuration

These properties are common across all Joint components:

| Property | Description |
|:---|:---|
| `Body` | The game object to find the target physics body to attach to. If null, attaches to the world. |
| `AnchorBody` | The body this joint is anchored to. If null, uses the GameObject this component is attached to. |
| `EnableCollision` | Whether the two connected bodies should collide with each other. |
| `StartBroken` | If true, the joint is broken and inactive when the scene starts. |
| `BreakForce` | The linear stress required to break the joint. |
| `BreakTorque` | The angular stress required to break the joint. |

### Spring Joint
Tries to keep an object a set distance away from another object, bouncing back like a spring. See the [Spring Joint Reference](../../scene/components/reference/springjoint.md) for full configuration details.

### Hinge Joint
Constrains objects to rotate around a single axis, like a door or a wheel.

| Property | Description |
|:---|:---|
| `MinAngle` | The minimum angle the hinge can rotate to. |
| `MaxAngle` | The maximum angle the hinge can rotate to. |
| `Friction` | Friction applied to resist rotation. |
| `TargetAngle` | If driving the joint with a motor, the angle it tries to reach. |
| `Frequency` | The stiffness of the motor. |
| `DampingRatio` | The damping of the motor. |
| `TargetVelocity` | If spinning the hinge as a motor, the speed it tries to reach. |
| `MaxTorque` | Maximum force the motor can apply to reach the target angle/velocity. |

### Slider Joint
Allows movement along a single line, like a piston.

| Property | Description |
|:---|:---|
| `MinLength` | Minimum distance along the slider axis. |
| `MaxLength` | Maximum distance along the slider axis. |
| `Friction` | Resistance to movement along the slider. |

### Ball Joint
Allows rotation in all directions around a single point, like a shoulder joint.

| Property | Description |
|:---|:---|
| `Friction` | Resistance to rotation. |
| `Frequency` | Stiffness of the joint if being driven. |
| `DampingRatio` | Damping of the joint if being driven. |
| `MaxTorque` | Maximum angular force the joint can apply. |

### Fixed Joint
Glues two objects together so they cannot move relative to each other.

| Property | Description |
|:---|:---|
| `LinearFrequency` | The linear stiffness of the connection. |
| `LinearDamping` | The linear damping of the connection. |
| `AngularFrequency` | The angular stiffness of the connection. |
| `AngularDamping` | The angular damping of the connection. |

## Troubleshooting

:::danger "My joint doesn't do anything!"
Make sure that both the object the Joint is attached to, and the target `Body` GameObject, have valid `Rigidbody` components. A joint needs physical bodies to constrain.
:::

:::warning "Bodies are spinning out of control"
If the two connected bodies intersect and `EnableCollision` is enabled, the physics engine will constantly try to push them apart while the joint tries to hold them together. Either space them apart or set `EnableCollision` to false.
:::

## Related Pages
- [Rigidbody](rigidbody.md)
- [Colliders](colliders.md)
