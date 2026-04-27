---
title: "Verlet Rope"
icon: "🔌"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/VerletRope.cs
updated: 2026-04-25
created: 2026-04-27
---

# Verlet Rope

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `VerletRope` component.

`VerletRope` is a physics-driven rope between two `GameObject` anchor points, simulated with Verlet integration. Unlike a `CableComponent` (which renders a cable through a chain of editor-positioned nodes), a `VerletRope` only knows about its start (`this.WorldPosition`) and an `Attachment` end — and the segments between are fully simulated, including gravity, collisions, stiffness, and damping.

## Quick Working Example

```csharp
var ropeGO = Scene.CreateObject();
ropeGO.WorldPosition = anchorTop.WorldPosition;

var rope = ropeGO.AddComponent<VerletRope>();
rope.Attachment = anchorBottom;
rope.SegmentCount = 24;
rope.Slack = 16f;

// Rope draws through a LineRenderer; auto-found if one is on the same GameObject
rope.LinkedRenderer = ropeGO.AddComponent<LineRenderer>();
```

## Properties

| Property | Description |
|:---|:---|
| `Attachment` | The `GameObject` the end of the rope is locked to. The rope re-initializes when this changes. |
| `LinkedRenderer` | The `LineRenderer` used to draw the rope. Falls back to `GetComponent<LineRenderer>()` on enable if unset. |
| `Slack` | Extra length added on top of the initial anchor-to-anchor distance. Clamped to `>= 0`. |
| `SegmentCount` | Number of segments. `1..64`. Higher = more accurate, much more expensive. Default `16`. |
| `Radius` | Sphere radius used for collision traces. Default `1`. |
| `LengthOverride` | If non-zero, replaces `initial length + Slack` entirely. Clamped to `>= 0`. |
| `Stiffness` | `0..1`+ — how strongly segments resist stretch. Default `0.7`. |
| `DampingFactor` | How quickly velocity decays per second. Default `0.2`. |
| `SoftBendFactor` | How strongly the rope resists bending. Default `0.3`. |

## Common Patterns

### Auto-resting

The rope detects when it's at rest (all non-attached points moving below a threshold for several consecutive frames) and stops simulating. It wakes back up automatically when either anchor moves more than 0.1 units, or every two seconds for a one-frame poll. This means a hanging rope is essentially free once it stops swinging.

### Driving rope length at runtime

```csharp
rope.LengthOverride = MathX.Lerp( rope.LengthOverride, target, Time.Delta * 4 );
```

`LengthOverride > 0` ignores `Slack` entirely, so use it when you want explicit control (winches, retractable lines). Setting it triggers `WakeRope()` so the rope is guaranteed to re-simulate.

### Pre-settled startup

`OnEnabled` runs 10 simulation steps immediately so a fresh rope appears already hanging in its rest pose rather than dropping into place visibly.

:::warning "Stretched rope skips collisions"
When the rope is stretched beyond `1.1×` its target length, per-segment collision checks are skipped on segments that move further than `1.7×` average segment length. Above `4×` stretch all collisions are disabled until the rope recovers — this is intentional, to stop a yanked rope from getting stuck inside world geometry.
:::

:::note "CableNodeComponent is internal"
The related `CableNodeComponent` (used by `CableComponent` for editor-placed cable waypoints) is `[Hide]`-marked and not intended for direct use — see [Cable Component](cablecomponent.md) for editor-driven cables.
:::

## Related Pages

- [Cable Component](cablecomponent.md)
- [Line Renderer](linerenderer.md)
