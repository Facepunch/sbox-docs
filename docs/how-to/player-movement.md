---
title: "Basic Player Movement"
icon: "🚶"
sources:
  - engine/Sandbox.Engine/Scene/Components/CharacterController/CharacterController.cs
updated: 2026-04-25
created: 2026-04-27
---

# Basic Player Movement

If you need a quick, copy-pasteable script to get a player character moving and jumping using the `CharacterController` component, use this recipe.

## Quick Working Example

1. Create a new GameObject for your player.
2. Add a `CharacterController` component to it.
3. Create a new C# script named `PlayerMovement.cs` and paste the code below.
4. Add the `PlayerMovement` component to your player GameObject.

```csharp
using Sandbox;

public sealed class PlayerMovement : Component
{
    // Automatically get the CharacterController component on this GameObject
    [RequireComponent] public CharacterController Controller { get; set; }

    [Property] public float Speed { get; set; } = 200f;
    [Property] public float JumpForce { get; set; } = 300f;
    [Property] public float Gravity { get; set; } = 800f;

    protected override void OnUpdate()
    {
        // 1. Get input direction
        var inputDirection = Input.AnalogMove;
        
        // 2. Build our desired velocity
        var wishVelocity = inputDirection * Speed;

        // 3. Accelerate towards our desired velocity smoothly
        Controller.Accelerate( wishVelocity );

        // 4. Apply gravity if we are in the air
        if ( !Controller.IsOnGround )
        {
            Controller.Velocity += Vector3.Down * Gravity * Time.Delta;
        }

        // 5. Jump if on the ground and requested
        if ( Controller.IsOnGround && Input.Pressed( "Jump" ) )
        {
            Controller.Punch( Vector3.Up * JumpForce );
        }

        // 6. Apply friction to slow down when there's no input
        Controller.ApplyFriction( 4.0f );

        // 7. Explicitly move the character
        Controller.Move();
    }
}
```

### Applying Gravity

Since `CharacterController` is kinematic, it does not fall automatically. You must manually apply downward velocity every frame the player is not touching the ground.

```csharp
if ( !Controller.IsOnGround )
{
    Controller.Velocity += Vector3.Down * Gravity * Time.Delta;
}
```

### Jumping

To make the character jump, you use the `Punch` method on the `CharacterController`. This adds an immediate burst of velocity.

```csharp
if ( Controller.IsOnGround && Input.Pressed( "Jump" ) )
{
    Controller.Punch( Vector3.Up * JumpForce );
}
```

### Crouching

To implement crouching, you adjust the `Height` property of the `CharacterController`. Be aware that reducing height typically shrinks the collider from the top down.

```csharp
[Property] public float StandingHeight { get; set; } = 64f;
[Property] public float CrouchingHeight { get; set; } = 32f;

protected override void OnUpdate()
{
    if ( Input.Down( "Crouch" ) )
    {
        Controller.Height = CrouchingHeight;
    }
    else
    {
        Controller.Height = StandingHeight;
    }
    
    // ... rest of your movement code
}
```

The `CharacterController` component provides several properties to adjust its collision hull and movement feel.

| Property | Description |
|---|---|
| `Radius` | The width of the character's collision cylinder. Default: `16.0f` |
| `Height` | The overall height of the character's collision cylinder. Default: `64.0f` |
| `StepHeight` | The maximum height of a ledge or step the character can automatically walk up without jumping. Default: `18.0f` |
| `GroundAngle` | The maximum slope angle (in degrees) the character can walk up. Steeper slopes will cause them to slide down. Default: `45.0f` |
| `Acceleration` | How quickly the character reaches their target velocity. Default: `10.0f` |
| `Bounciness` | A value from 0 to 1 determining how much the character bounces off walls when jumping into them. Default: `0.3f` |

## Troubleshooting

:::danger "My character is sinking into the floor!"
Ensure the `Height` and `Radius` properties on your `CharacterController` match the visual scale of your player model so they don't appear to float or sink into the ground. Remember that changing the model's scale does not automatically scale the controller's hull.
:::

:::warning "Character stops completely on small bumps!"
If your character cannot walk over small obstacles or stairs, try increasing the `StepHeight` property. It must be higher than the obstacle you are trying to step over.
:::

## Related Pages
- [Player Controller Reference](../scene/components/reference/player-controller.md) (For a complete, built-in solution)
- [Detect Player Input](detect-input.md)
- [Camera Follow Player](camera-follow.md) — pair this movement with a third-person follow camera.
- [Apply Physics Forces](apply-physics-forces.md) — use forces instead when you want a `Rigidbody`-driven character.
