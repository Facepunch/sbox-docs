---
title: "Camera Follow Player"
icon: "📹"
sources:
  - engine/Sandbox.Engine/Scene/Components/Camera/CameraComponent.cs
updated: 2026-04-25
created: 2026-04-27
---

# Camera Follow Player

How to make the main scene camera smoothly follow a player or object.

## Quick Working Example

```csharp
using Sandbox;

public sealed class CameraFollow : Component
{
    [Property] public GameObject Target { get; set; }
    [Property] public Vector3 Offset { get; set; } = new Vector3( -300, 0, 150 );
    [Property] public float SmoothSpeed { get; set; } = 5f;

    protected override void OnUpdate()
    {
        if ( !Target.IsValid() ) return;

        // Calculate the desired position based on the target's position and our offset
        var targetPosition = Target.WorldPosition + Offset;

        // Smoothly interpolate the camera's position towards the target position
        GameObject.WorldPosition = Vector3.Lerp( GameObject.WorldPosition, targetPosition, Time.Delta * SmoothSpeed );

        // Always look at the target
        var lookDirection = Target.WorldPosition - GameObject.WorldPosition;
        GameObject.WorldRotation = Rotation.LookAt( lookDirection, Vector3.Up );
    }
}
```

### Following with a Fixed Offset

The example above uses a fixed world-space offset. This means the camera always stays "North" of the player, which is perfect for top-down or isometric games.

### Following Behind the Player (Third Person)

If you want the camera to always be directly *behind* the player (e.g., an over-the-shoulder view or racing game), you need to calculate the offset relative to the target's rotation.

```csharp
protected override void OnUpdate()
{
    if ( !Target.IsValid() ) return;

    // Use the target's rotation to determine what "behind" and "up" mean
    var backward = Target.WorldRotation.Backward;
    var up = Target.WorldRotation.Up;

    // Calculate a position 200 units behind and 100 units above the target
    var targetPosition = Target.WorldPosition + (backward * 200f) + (up * 100f);

    GameObject.WorldPosition = Vector3.Lerp( GameObject.WorldPosition, targetPosition, Time.Delta * SmoothSpeed );
    GameObject.WorldRotation = Rotation.LookAt( Target.WorldPosition - GameObject.WorldPosition, Vector3.Up );
}
```

## Troubleshooting

:::warning "The camera stutters or jitters when moving!"
If the target is moving using Physics (e.g., a Rigidbody) in `OnFixedUpdate`, but the camera is following it in `OnUpdate`, you may see stuttering due to the differing framerates. Try changing the camera follow script to use `OnPreRender` or ensure physics interpolation is enabled on the target.
:::

:::danger "Null Reference Exception on Target"
A frequent error is forgetting to assign the `Target` property in the Editor Inspector. Ensure you drag your player GameObject into the `Target` field before pressing Play. Always check `Target.IsValid()` instead of `Target == null` for GameObjects in s&box, as destroyed objects may not be strictly null immediately.
:::

## Related Pages
- [Camera Component](../scene/components/reference/cameracomponent.md)
- [Rotate an Object](rotate-object.md)
- [Basic Player Movement](player-movement.md) — what you'll typically be following with this camera.
- [Detect Player Input](detect-input.md) — pair with mouse-look input for a free-look third-person view.
