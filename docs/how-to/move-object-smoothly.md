---
title: "Move an Object Smoothly"
icon: "🎬"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.cs
created: 2026-04-27
updated: 2026-04-27
---

# Move an Object Smoothly

How to smoothly move a GameObject towards a target position using interpolation (`Lerp`) or damping (`SmoothDamp`).

## Quick Working Example

```csharp
using Sandbox;

public sealed class FollowTarget : Component
{
    [Property] public GameObject Target { get; set; }
    [Property] public float FollowSpeed { get; set; } = 5f;

    protected override void OnUpdate()
    {
        if ( Target == null ) return;

        // Smoothly move our position towards the target's position
        GameObject.WorldPosition = Vector3.Lerp( GameObject.WorldPosition, Target.WorldPosition, Time.Delta * FollowSpeed );
    }
}
```

### Using Vector3.Lerp

`Vector3.Lerp` is the simplest way to smoothly follow a moving target. It takes the current position, the target position, and a fraction (0.0 to 1.0) indicating how far to move this frame.

```csharp
protected override void OnUpdate()
{
    // A speed of 5.0 means we move 500% of the distance per second, 
    // capped by the frame time.
    float speed = 5.0f;
    GameObject.LocalPosition = Vector3.Lerp( GameObject.LocalPosition, TargetPosition, Time.Delta * speed );
}
```

### Using Vector3.SmoothDamp

If you need more realistic physical movement without overshoot, use `SmoothDamp`. It requires you to store the current velocity as a `ref` parameter.

```csharp
public class SmoothFollower : Component
{
    [Property] public GameObject Target { get; set; }
    
    // We must keep track of the current velocity between frames
    private Vector3 _currentVelocity;

    protected override void OnUpdate()
    {
        if ( Target == null ) return;

        // The time it should take to reach the target (lower is faster)
        float smoothTime = 0.3f;

        GameObject.WorldPosition = Vector3.SmoothDamp( 
            GameObject.WorldPosition, 
            Target.WorldPosition, 
            ref _currentVelocity, 
            smoothTime, 
            Time.Delta 
        );
    }
}
```

### Interpolating Single Values (MathX.LerpTo)

If you only need to smooth a single number (like a `float` for alpha or a rotation angle) instead of a full `Vector3`, use `MathX.LerpTo` or `MathX.Lerp`.

```csharp
[Property] public float TargetAlpha { get; set; } = 1f;
private float _currentAlpha = 0f;

protected override void OnUpdate()
{
    // Smoothly transition our float to the target value
    _currentAlpha = MathX.LerpTo( _currentAlpha, TargetAlpha, Time.Delta * 10f );
}
```

## Troubleshooting

:::danger "My object moves at different speeds depending on framerate!"
Always multiply your interpolation speed by `Time.Delta`. If you just use a fixed number like `Vector3.Lerp(a, b, 0.1f)`, the object will move much faster at 144hz than at 60hz.
:::

:::warning "The object never fully reaches the target!"
Because `Lerp` moves a fraction of the *remaining* distance, it technically never reaches 100%. If you need an object to perfectly snap to the end after it gets close enough, check the distance:

```csharp
if ( Vector3.DistanceBetween( GameObject.WorldPosition, TargetPosition ) < 0.1f )
{
    GameObject.WorldPosition = TargetPosition; // Snap to the end
}
```
:::

## Related Pages
- [Rotate an Object](rotate-object.md)
- [Camera Follow Player](camera-follow.md)
