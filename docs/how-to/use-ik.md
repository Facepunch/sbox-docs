---
title: "Use Inverse Kinematics (IK)"
icon: "♿"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/SkinnedModelRenderer.Parameters.cs
created: 2026-04-27
updated: 2026-04-27
---

# Use Inverse Kinematics (IK)

How to set up and use Inverse Kinematics (IK) on an animated model to make hands or feet dynamically reach for targets in the world.

## Quick Working Example

```csharp
using Sandbox;

public sealed class IKHandler : Component
{
    [Property] public SkinnedModelRenderer Renderer { get; set; }
    [Property] public GameObject TargetObject { get; set; }

    protected override void OnUpdate()
    {
        if ( Renderer == null || TargetObject == null ) return;

        // Pass the world transform of our target to the "hand_r" IK parameter
        Renderer.SetIk( "hand_r", TargetObject.WorldTransform );
    }
}
```

### Setting IK Targets

You can update the IK target every frame in `OnUpdate()`. The `Transform` you provide should be in World Space. The engine will automatically convert it to the model's local space before passing it to the animation graph.

```csharp
public void UpdateFootIK( Transform leftFootTarget, Transform rightFootTarget )
{
    Renderer.SetIk( "foot_left", leftFootTarget );
    Renderer.SetIk( "foot_right", rightFootTarget );
}
```

### Clearing IK Targets

When the character stops interacting with the object (e.g., lets go of the door handle), you must disable the IK so the normal animation takes over again. Use `ClearIk()`.

```csharp
public void ReleaseHandle()
{
    // Disables the IK, returning control to the standard animation
    Renderer.ClearIk( "hand_r" );
}
```

## Troubleshooting

:::danger "The limb snaps back and forth wildly!"
Ensure you are passing a **World Space** transform to `SetIk()`. The method expects world coordinates and will handle the conversion to local space internally. If you pass a local transform, it will calculate incorrectly relative to the model's root.
:::

:::warning "Nothing happens when I call SetIk!"
The `SetIk` method only sets parameters in the Animation Graph. Your model's Animation Graph must be configured to actually *use* those parameters. 

It needs an IK node (like Two-Bone IK) that reads the `ik.hand_r.position` and `ik.hand_r.rotation` parameters, and blends based on `ik.hand_r.enabled`. If the graph doesn't have this setup, the code will do nothing.
:::

## Related Pages
- [Skinned Model Renderer](../scene/components/reference/skinnedmodelrenderer.md)
