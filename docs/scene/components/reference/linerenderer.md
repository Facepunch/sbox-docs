---
title: "Line Renderer"
icon: "📈"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/LineRenderer.cs
updated: 2026-04-25
created: 2026-04-27
---

# Line Renderer

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `LineRenderer` component.

The Line Renderer component draws a continuous 3D line connecting a specific list of points. It is useful for drawing paths, laser tripwires, grapnel cables, or visualizing navmesh routes.

## Quick Working Example

```csharp
public class LaserTripwire : Component
{
	[Property] public LineRenderer Laser { get; set; }
	[Property] public GameObject StartPoint { get; set; }
	[Property] public GameObject EndPoint { get; set; }

	protected override void OnStart()
	{
		if ( !Laser.IsValid() ) return;

		// Feed the GameObjects directly to the LineRenderer
		Laser.UseVectorPoints = false;
		Laser.Points = new List<GameObject> { StartPoint, EndPoint };
		
		// Configure appearance
		Laser.Color = Color.Red;
		Laser.Width = 2f;
	}
}
```

## Configuration

| Group | Property | Description |
|---|---|---|
| Points | `UseVectorPoints` | If true, the renderer uses a list of `Vector3` coordinates. If false, it uses a list of `GameObject` transforms. |
| Points | `Points` / `VectorPoints` | The list of points to connect. |
| Appearance | `Face` | How the line geometry faces the camera (e.g., `Camera` to always face the view, or `Cylinder` for fully 3D tubes). |
| Appearance | `Color` / `Width` | Gradients and curves applied across the length of the entire line. |
| Spline | `SplineInterpolation` | Adds additional subdivisions to smooth curves between points. A value of 1 means straight lines. |
| Spline | `SplineTension` / `Continuity` / `Bias` | Tweak these values to adjust how the spline curves through the points. |
| End Caps | `StartCap` / `EndCap` | The style of the ends of the line (e.g., Flat, Rounded). |
| Rendering | `Lighting` | If true, the line will be affected by scene lighting. |

## Troubleshooting

:::warning Disappearing Lines
If your line renderer uses GameObjects as points, remember that if any of those GameObjects are disabled or destroyed, the point is skipped. If fewer than 2 valid points remain, the entire line will stop rendering.
:::

:::danger Cylinder Face Mode Limitations
When setting `Face` to `Cylinder` to draw 3D pipes/tubes, the `StartCap` and `EndCap` settings are not supported.
:::

## Related Pages
- [Trail Renderer](trailrenderer.md)
- [Cable Component](cablecomponent.md)
