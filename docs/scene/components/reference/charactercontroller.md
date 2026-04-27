---
title: "Character Controller"
icon: "🚶"
sources:
  - engine/Sandbox.Engine/Scene/Components/CharacterController/CharacterController.cs
updated: 2026-04-25
created: 2026-04-27
---

# Character Controller

> **New to physics?** Physics overview is at **[Physics](../../../systems/physics/index.md)**. This page is the reference for the `CharacterController` component.

The `CharacterController` component provides collision-constrained movement without using a Rigidbody. It is not affected by physical forces and only moves when you explicitly tell it to via code, making it ideal for player characters or specialized AI.

## Quick Working Example

```csharp
public class CustomPlayerMovement : Component
{
    [RequireComponent] CharacterController Controller { get; set; }

    protected override void OnUpdate()
    {
        // 1. Build wish velocity based on input
        var wishVelocity = Input.AnalogMove * 200f;

        // 2. Accelerate smoothly towards the wish velocity
        Controller.Accelerate( wishVelocity );

        // 3. Apply gravity if not on the ground
        if ( !Controller.IsOnGround )
        {
            Controller.Velocity += Vector3.Down * 800f * Time.Delta;
        }

        // 4. Jump if on the ground and requested
        if ( Controller.IsOnGround && Input.Pressed( "Jump" ) )
        {
            Controller.Punch( Vector3.Up * 300f );
        }

        // 5. Apply friction to slow down
        Controller.ApplyFriction( 4.0f );

        // 6. Explicitly move the character
        Controller.Move();
    }
}
```

### Basic Movement

To move the character, you update its `Velocity` property and then call `Move()`. The system provides helper methods like `Accelerate` and `ApplyFriction` to make movement feel natural without needing to scale by `Time.Delta` yourself.

```csharp
// Smoothly speed up in a direction
Controller.Accelerate( Vector3.Forward * 100f );

// Slow down to a stop (frictionAmount, optional stopSpeed threshold)
Controller.ApplyFriction( 5.0f, 140.0f );

// Apply the actual movement
Controller.Move();
```

### Jumping

To make the character jump, you break its connection to the ground and add upward velocity using `Punch`.

```csharp
if ( Controller.IsOnGround && Input.Pressed( "Jump" ) )
{
    Controller.Punch( Vector3.Up * 300f );
}
```

### AI or Scripted Movement

For NPCs or cinematic sequences, you often want to move the character to a specific location while still respecting ground and wall collisions. You can use the `MoveTo` method.

```csharp
public class AINavigator : Component
{
    [RequireComponent] public CharacterController Controller { get; set; }

    protected override void OnFixedUpdate()
    {
        Vector3 targetPosition = new Vector3( 100, 100, 0 );
        
        // Move towards the target position, passing true to allow stepping over obstacles
        Controller.MoveTo( targetPosition, useStep: true );
    }
}
```

### Tracing

Sometimes you need to know if the character would hit something if it moved in a certain direction without actually moving it. You can use `TraceDirection` for this. This uses the character's bounding box and collision rules to trace a path.

```csharp
// Trace 50 units downward to check for a drop
var traceResult = Controller.TraceDirection( Vector3.Down * 50f );

if ( traceResult.Hit )
{
    Log.Info( $"We would hit {traceResult.GameObject.Name}" );
}
```

## Configuration

| Property | Description |
|---|---|
| **Radius** | The radius of the character's bounding cylinder. |
| **Height** | The total height of the character's bounding cylinder. |
| **StepHeight** | The maximum height of an obstacle the character can step over automatically. |
| **GroundAngle** | The maximum slope angle (in degrees) the character can stand on. |
| **Acceleration** | Determines how quickly `Accelerate` reaches the target velocity. |
| **Bounciness** | How much the controller bounces off walls (0 to 1). |
| **UseCollisionRules**| Determine what to collide with using the project's collision rules for the GameObject's Tags. |
| **IgnoreLayers** | TagSet of layers to ignore if `UseCollisionRules` is false. |
| **Velocity** | The character's current movement velocity. You can read or modify this directly. |
| **IsOnGround** | (Read-only) True if the character is currently standing on valid ground. |
| **GroundObject** | (Read-only) The GameObject the character is standing on. |
| **GroundCollider** | (Read-only) The Collider the character is standing on. |

## Troubleshooting

:::danger "My character isn't moving!"
A `CharacterController` is **not** a `Rigidbody`. Applying forces via `Rigidbody.ApplyForce` will do nothing. You **must** explicitly call `Controller.Move()` or `Controller.MoveTo()` inside your `OnUpdate` or `OnFixedUpdate` loop for the position to update.
:::

:::warning "My character is floating/sinking!"
Ensure the `Height` and `Radius` properties match the visual scale of your `ModelRenderer`. The controller uses a cylinder for its collision shape, and if it's too tall, the character will appear to float.
:::

## Related Pages
- [Player Controller](player-controller.md) (A higher-level, ready-to-use controller built on top of this system)
- [Physics / Rigidbody](../../../systems/physics/rigidbody.md)
- [Tracing](../../scenes/tracing.md)
