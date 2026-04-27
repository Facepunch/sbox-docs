---
title: "Menu VR Support"
icon: "🥽"
sources:
  - game/addons/menu/Code/VR/
created: 2026-04-27
updated: 2026-04-27
---

# Menu VR Support

The Menu addon contains helper components to adapt its 3D background scene and interaction models when the player launches the engine in Virtual Reality (VR).

## Quick Start Example

If you want your custom menu scene to react differently when the user puts on a headset, you can use the `VRSceneSwitcher`:

```csharp
using Sandbox;
using Menu;

public sealed class VRAwareComponent : Component
{
    protected override void OnStart()
    {
        if ( Game.IsRunningInVR )
        {
            Log.Info( "User is in VR. Instantiating 3D spatial UI pointers." );
        }
    }
}
```

## Common Patterns

1. **Dual Scenes:** Instead of trying to write a single scene that perfectly accommodates both desktop and VR, the menu addon pattern is to create two distinct scenes and use `VRSceneSwitcher` to seamlessly route the player on startup.
2. **Input Adapters:** In a desktop menu, the mouse clicks standard Razor elements. In a VR menu, a spatial laser pointer (`CursorPresser`) usually casts a ray against a `WorldPanel` to simulate mouse clicks.

## Troubleshooting

:::warning Common Gotchas
- **UI is glued to my face in VR:** If you launch in VR and the menu behaves like a desktop game, ensure `VRSceneSwitcher` is present in your default startup scene and properly points to a VR-configured target scene.
- **Can't click anything:** Razor `ScreenPanel` UI does not receive physical VR pointer clicks natively. You must use a `WorldPanel` placed in 3D space.
:::

## Related Pages
- [Menu Architecture](./menu-architecture.md)

