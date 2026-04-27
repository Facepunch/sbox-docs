---
title: "VR Overlays"
sources:
  - engine/Sandbox.Engine/Platform/VR/Overlay/
updated: 2026-04-27
created: 2026-04-27
---

# VR Overlays

The VR Overlay system allows you to render 2D UI panels directly into the VR environment. These overlays exist independent of the 3D scene geometry and are composited by the VR runtime itself, making them perfect for crisp, high-quality menus, HUDs, or notifications.

## Quick Start

```csharp
using Sandbox;

public sealed class VROverlayMenu : Component
{
    private VROverlay _menuOverlay;

    protected override void OnStart()
    {
        if ( !VR.IsActive ) return;

        // Create a new overlay panel
        _menuOverlay = new VROverlay( "MainMenu", "Main Menu" );
        
        // Set its size (in inches) and position in the world
        _menuOverlay.Width = 59.0f; // Roughly 1.5 meters
        _menuOverlay.Transform = new Transform( new Vector3( 0, 50, 50 ), Rotation.Identity );
        
        // Render a UI panel to it
        // (Assuming you have a standard UI panel setup)
        // _menuOverlay.Texture = myPanel.RenderToTexture(); 
        
        _menuOverlay.Show();
    }

    protected override void OnDestroy()
    {
        _menuOverlay?.Destroy();
    }
}
```

## Common Patterns

### HUD Attachments
You can parent overlays to the user's head or controllers, ensuring the UI follows them as they move.

```csharp
protected override void OnUpdate()
{
    if ( _menuOverlay != null && VR.IsActive )
    {
        // Keep the menu 1 meter in front of the player's face
        var headPos = VR.Head.Position;
        var forward = VR.Head.Rotation.Forward;
        
        _menuOverlay.Transform = new Transform( headPos + (forward * 50f), VR.Head.Rotation );
    }
}
```

## Troubleshooting

:::warning Z-Ordering and Clipping
Because overlays are composited *after* the scene is rendered, they will draw on top of everything, even if they should technically be blocked by a wall or an object in the 3D world. Use them carefully for HUDs or menus, but avoid using them for UI elements that belong diegetically inside the game world.
:::

## Related Pages
* [VR Platform Integration](./vr.md)
* [UI System](../../systems/ui/index.md)
