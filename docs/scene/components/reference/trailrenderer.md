---
title: "Trail Renderer"
icon: "📈"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/TrailRenderer.cs
  - engine/Sandbox.Engine/Systems/SceneSystem/SceneTrailObject.cs
updated: 2026-04-25
created: 2026-04-27
---

# Trail Renderer

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `TrailRenderer` component.

The Trail Renderer component automatically draws a continuous ribbon or trail behind a moving `GameObject`. It is perfect for weapon swings, tire tracks, missile smoke trails, and Tron-style light cycles.

## Quick Working Example

```csharp
public class SwordSwingTrail : Component
{
	[Property] public TrailRenderer Trail { get; set; }

	// Called via animation events or attack logic
	public void StartSwing()
	{
		Trail.Emitting = true;
	}

	public void EndSwing()
	{
		Trail.Emitting = false;
	}
}
```

## Configuration

| Group | Property | Description |
|---|---|---|
| Trail | `MaxPoints` | The maximum number of points to keep in the trail. |
| Trail | `PointDistance` | The minimum distance the object must move before a new point is added. |
| Trail | `LifeTime` | How long (in seconds) each point lasts before fading out or disappearing. |
| Trail | `Emitting` | When enabled, new points are added as the object moves. |
| Appearance | `Texturing` | Controls the material and how UV coordinates are stretched or repeated across the trail. |
| Appearance | `Color` | A color gradient applied along the length of the trail. |
| Appearance | `Width` | A curve controlling the thickness of the trail along its length. |
| Appearance | `Face` | How the trail geometry faces the camera (e.g., `Camera` to always face the view). |
| Rendering | `Opaque` | If true, the trail is rendered opaque and can cast shadows. If false, it uses `BlendMode`. |

## Troubleshooting

:::warning Choppy Trails
If your trail looks jagged or segmented, try lowering the `PointDistance`. This forces the renderer to add points more frequently as the object moves, smoothing out sharp corners. Keep in mind this may require increasing `MaxPoints` if the trail is very long.
:::

:::tip Disconnected Trails
If you teleport an object with an active Trail Renderer, it will draw a straight line from the old position to the new one. To avoid this, temporarily disable `Emitting` before teleporting, or reset the trail entirely.
:::

## Related Pages
- [Line Renderer](linerenderer.md)
- [Particle Effect](../../../systems/effects/particle-effect/index.md)
