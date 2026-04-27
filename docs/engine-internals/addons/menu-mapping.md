---
title: "Menu Mapping Components"
icon: "🗺️"
sources:
  - game/addons/menu/Code/Map/
created: 2026-04-27
updated: 2026-04-27
---

# Menu Mapping Components

The Menu addon includes a `Sandbox.Mapping` namespace that provides foundational interactive components intended for use within scenes, particularly maps that feature diegetic UI or interactive world elements.

## Quick Start Example

You can add interactive buttons to your scene that react to player clicks or physical touches.

```csharp
using Sandbox;
using Sandbox.Mapping;

public sealed class ButtonWatcher : Component
{
    [Property] public Button TargetButton { get; set; }

    protected override void OnStart()
    {
        if ( TargetButton != null )
        {
            // Subscribe to the button's C# Action
            TargetButton.OnStateChanged += HandleButtonState;
        }
    }

    private void HandleButtonState( bool isOn )
    {
        Log.Info( $"The map button was turned {(isOn ? "ON" : "OFF")}!" );
    }
}
```

## Included Components

1. **Button:** A highly configurable interactive trigger. It supports three modes:
   - *Toggle:* Click to turn on, click again to turn off. (Can auto-reset after a delay).
   - *Continuous:* Only stays "On" while the player is actively holding the interact button.
   - *Immediate:* Fires an "On" event and immediately resets in a single frame.
   The button can also be configured to physically move (translate along a vector) using an `AnimationCurve` when pressed.

2. **LinearMove:** A utility component for smoothly translating a GameObject from point A to point B over time, often driven by a `Button`'s `OnTurnedOn` event.

3. **CursorPresser:** A helper component designed to attach to a player's camera or a VR controller. It projects a raycast forward, detects objects implementing `IPressable`, and passes click/release events to them.

## Common Patterns

1. **Diegetic Menus:** If your game uses a 3D physical world menu (like a hub room where players shoot a physical start button), you would use `CursorPresser` on the player and `Button` on the targets.
2. **Event Routing:** Rather than writing a custom script for every door or switch, use the `Button` component's `OnStateChanged` Action in the Inspector to wire up connections to other scripts visually, similar to Unity Events.

## Troubleshooting

:::warning Common Gotchas
- **Button isn't clickable:** Ensure the GameObject with the `Button` component also has a physics `Collider`. The `CursorPresser` relies on physics raycasts to find the `IPressable` interface.
- **Button doesn't animate:** Ensure `Move` is checked in the Inspector, a `MoveDelta` is defined, and that the `AnimationTime` is greater than 0.
:::

## Related Pages
- [Trace / Raycast](../../systems/physics/physics-events.md)

