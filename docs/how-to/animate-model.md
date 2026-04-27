---
title: "Animate a Model"
icon: "🏃"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/SkinnedModelRenderer.cs
  - engine/Sandbox.Engine/Systems/Render/AnimGraphDirectPlayback.cs
created: 2026-04-27
updated: 2026-04-27
---

# Animate a Model

How to play animations or trigger states on a skeletal model using `SkinnedModelRenderer`.

## Quick Working Example

```csharp
using Sandbox;

public sealed class PlayerAnimator : Component
{
    // The SkinnedModelRenderer attached to our player model
    [RequireComponent] public SkinnedModelRenderer Renderer { get; set; }

    protected override void OnUpdate()
    {
        // Example 1: Setting a parameter in the Animation Graph
        // This tells the AnimGraph to transition to the running state
        Renderer.Set( "b_run", true );
        
        // Example 2: Setting a float to blend animation speeds
        Renderer.Set( "move_speed", 250f );

        // Example 3: Playing an animation directly (bypassing the AnimGraph logic)
        if ( Input.Pressed( "attack1" ) )
        {
            Renderer.SceneModel.DirectPlayback.Play( "attack" );
        }
    }
}
```

### Driving an Animation Graph

If your model has an AnimGraph attached, it is expecting variables to tell it what to do. You pass these variables from your C# code using `Renderer.Set()`.

```csharp
// Booleans are great for state toggles
Renderer.Set( "b_grounded", Controller.IsOnGround );
Renderer.Set( "b_crouch", Input.Down( "Crouch" ) );

// Floats are great for blending (e.g., walking vs running)
Renderer.Set( "move_direction", 90f );

// You can also pass Vectors or Rotations if your AnimGraph uses them
Renderer.Set( "look_target", TargetGameObject.WorldPosition );
```

### Playing a Sequence Directly

Sometimes you just want to play an animation from start to finish. You can access the `DirectPlayback` system through the underlying `SceneModel`.

```csharp
public void PlayDeathAnimation()
{
    // Plays the sequence named "die" and stops other animations
    Renderer.SceneModel.DirectPlayback.Play( "die" );
}

public void CancelAnimation()
{
    // Stops the direct playback, returning control to the AnimGraph
    Renderer.SceneModel.DirectPlayback.Cancel();
}
```

## Configuration

| Property | Description |
|:---|:---|
| `UseAnimGraph` | (In the Editor) Whether to evaluate the animation graph attached to the model. If you only want to use Direct Playback, you can disable this. |
| `Renderer.SceneModel.DirectPlayback.TimeNormalized` | Gets the normalized time (0.0 to 1.0) of the currently playing direct sequence. |
| `Renderer.SceneModel.DirectPlayback.Duration` | Gets the duration (in seconds) of the currently playing direct sequence. |

## Troubleshooting

:::danger "My parameters aren't doing anything!"
1. Verify `UseAnimGraph` is checked on the `SkinnedModelRenderer`.
2. Open the model in the Model Editor and verify it actually has an Animation Graph assigned to it.
3. Ensure the parameter name you are passing exactly matches the parameter name inside the AnimGraph.
:::

:::warning "Direct Playback won't play my animation!"
The name passed to `DirectPlayback.Play()` must be the name of a **Sequence** compiled into the model, not an AnimGraph state. You can view all valid sequences by opening the model in the Model Editor and looking at the Animation list.
:::

## Related Pages
- [Skinned Model Renderer](../scene/components/reference/skinnedmodelrenderer.md)
- [Use Inverse Kinematics (IK)](use-ik.md)
