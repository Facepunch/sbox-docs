---
title: "Chair"
icon: "🪑"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/BaseChair.cs
updated: 2026-04-25
created: 2026-04-27
---

# Chair

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `BaseChair` component.

`BaseChair` makes a `GameObject` sittable by a `PlayerController`. It implements `IPressable`, so a player can press the use key on the chair to enter, and the component then re-parents the player into the seat, locks their body, applies a sit pose, clamps their look angles, and provides an exit point on departure. Subclass it to customise enter/leave behavior.

## Quick Working Example

```csharp
var chair = chairProp.AddComponent<BaseChair>();
chair.SeatPosition = chairProp.Children.FirstOrDefault( x => x.Name == "seat" );
chair.EyePosition = chairProp.Children.FirstOrDefault( x => x.Name == "eye" );
chair.SitPose = BaseChair.AnimatorSitPose.Chair;
chair.PitchRange = new Vector2( -45, 45 );
chair.YawRange = new Vector2( -90, 90 );
```

A player walking up to it will see a "Sit" tooltip and can press use to sit down.

## Properties

| Property | Description |
|:---|:---|
| `SeatPosition` | `GameObject` whose transform defines where the player is placed. Falls back to this `GameObject`. |
| `SitPose` | `AnimatorSitPose` enum (`Chair`, `ChairForward`, `ChairCrossed`, `Kneeling`, `Ground`, ...). Drives the `sit` animator parameter. Default `Chair`. |
| `SitHeight` | Per-pose height adjust, `-1..1`. Multiplied by 12 and written to `sit_offset_height`. Default `0`. |
| `EyePosition` | `GameObject` whose transform defines the seated eye position. Falls back to `SeatPosition`. |
| `PitchRange` | Min/max pitch (degrees) the seated player can look. Default `(-90, 70)`. |
| `YawRange` | Min/max yaw (degrees) the seated player can look. Default `(-120, 120)`. |
| `ExitPoints` | Array of `GameObject`s the player can be ejected to. The one most aligned with the player's look direction is chosen. |
| `TooltipTitle` | Tooltip title shown when hovering. Default `"Sit"`. Empty disables the tooltip. |
| `TooltipIcon` | Material Icons identifier or emoji. Default `"airline_seat_recline_normal"`. |
| `TooltipDescription` | Longer description for the tooltip. |

## Common Patterns

### Custom enter/leave conditions

```csharp
public class DriverSeat : BaseChair
{
	public override bool CanLeave( PlayerController player )
		=> Vehicle.Speed < 5f;
}
```

### Multiple exit points

If the chair has obstructions on one side (a wall behind), populate `ExitPoints` with `GameObject`s representing valid exit positions. The component picks whichever one the player is most facing toward.

:::warning "Eye transform requires PostCameraSetup wiring"
The seated eye transform is enforced via `CalculateEyeTransform` only if your camera/player code reads from it. Standard player controllers do this automatically, but a custom camera may need to call `chair.CalculateEyeTransform( player )` itself.
:::

:::note "Re-parenting is RPC-broadcast"
Because the player is re-parented client-side via `Sit`/`Eject` broadcasts (rather than host-side replication), each client owns its own seated state. This is a deliberate workaround for replicated parenting.
:::

## Related Pages

- [Player Controller](player-controller.md)
- [Component Events](../events/index.md)
