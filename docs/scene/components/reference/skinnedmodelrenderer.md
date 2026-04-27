---
title: "Skinned Model Renderer"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/SkinnedModelRenderer.cs
updated: 2026-04-25
created: 2026-04-27
---

# Skinned Model Renderer

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `SkinnedModelRenderer` component.

The `SkinnedModelRenderer` component is similar to `ModelRenderer`, but it is meant for animated models that have a skeletal structure (bones), such as characters or creatures.

## Quick Working Example

```csharp
public class CharacterAnimator : Component
{
	[RequireComponent] SkinnedModelRenderer Renderer { get; set; }

	protected override void OnUpdate()
	{
		// Tell the animator to transition to a running state if we're moving fast
		Renderer.Set( "b_run", true );
		
		// Set a float parameter to blend animations based on speed
		Renderer.Set( "move_speed", 250f );
	}
}
```

### Setting Animation Parameters

Most modern games use an animation graph (or state machine) to blend between animations. You can drive this graph by passing parameters into the renderer.

```csharp
var renderer = Components.Get<SkinnedModelRenderer>();

// Set a boolean parameter
renderer.Set( "b_is_jumping", true );

// Set a float parameter (e.g. blend weight or movement speed)
renderer.Set( "move_direction", 90f );

// Set an integer parameter
renderer.Set( "weapon_type", 2 );
```

### Direct Playback (Sequences)

If you have a simple animated object (like a spinning coin or opening chest) and don't want to build a full animation graph, you can play a sequence directly.

```csharp
var renderer = Components.Get<SkinnedModelRenderer>();

// Play a single animation sequence by name, looping indefinitely
renderer.DirectPlayback.Play( "open_chest" );
```

### Parenting to Bones

You can retrieve bone transforms to attach objects like weapons to hands, or hats to heads.

```csharp
var renderer = Components.Get<SkinnedModelRenderer>();
var rightHand = renderer.GetBoneObject( "hold_R" );

if ( rightHand.IsValid() )
{
	myWeapon.SetParent( rightHand );
	myWeapon.LocalPosition = Vector3.Zero;
}
```

### Inverse Kinematics (IK)

If the model's animation graph is set up to accept IK targets, you can dynamically override limb positions (like making a hand reach for a doorknob). The renderer exposes a quick helper method for this.

```csharp
var renderer = Components.Get<SkinnedModelRenderer>();

// Tell the animation graph to move the "hand_r" IK target to this world transform
renderer.SetIk( "hand_r", targetTransform );

// Disable the IK target, returning control to the standard animation
renderer.ClearIk( "hand_r" );
```

See the [Use Inverse Kinematics (IK)](../../../how-to/use-ik.md) guide for more details.

## Configuration

| Property | Description |
|:---|:---|
| `Model` | The skeletal `Model` resource to render. |
| `UseAnimGraph` | Whether to evaluate the animation graph attached to the model. |
| `AnimationGraph` | Override the model's default animation graph. |
| `PlaybackRate` | Multiplier for animation playback speed (0.0 - 4.0). |
| `CreateBoneObjects` | Spawn child GameObjects for each bone so they can be referenced or parented to. |
| `BoneMergeTarget` | Another `SkinnedModelRenderer` whose bones this model should follow (e.g. for clothing on a character). |

## Troubleshooting

:::warning "My character is T-Posing!"
1. Verify `UseAnimGraph` is checked.
2. Verify the model actually has an Animation Graph assigned to it in the asset settings.
3. Ensure you are sending the correct parameter names that the animation graph expects.
:::

## Related Pages
- [Model Renderer](modelrenderer.md)
