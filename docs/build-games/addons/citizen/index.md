---
title: Citizen Addon
sources:
  - game/addons/citizen
created: 2026-04-27
updated: 2026-04-27
---

# Citizen Addon

The `citizen` addon, located at `game/addons/citizen/`, contains the default "Citizen" player model, its animations, and the core clothing system used across many s&box games.

## Quick Working Example

You can spawn the Citizen model in your game by referencing its compiled `.vmdl` path provided by this addon, and using the `CitizenAnimationHelper` to drive it.

```csharp
using Sandbox;

public sealed class PlayerSpawner : Component
{
    protected override void OnStart()
    {
        // Create an empty GameObject
        var playerObj = new GameObject(true, "Player");
        
        // Add a SkinnedModelRenderer to draw the animated model
        var renderer = playerObj.Components.Create<SkinnedModelRenderer>();
        renderer.Model = Model.Load( "models/citizen/citizen.vmdl" );
        
        // Add the helper to manage the complex animation graph
        var animHelper = playerObj.Components.Create<CitizenAnimationHelper>();
        
        // Now you can drive animations easily
        animHelper.WithVelocity( Vector3.Forward * 100f );
    }
}
```

1. **Prototyping:** Use the Citizen model to get your player controller working immediately before your custom art is ready.
2. **Standardization:** By using the Citizen skeleton, your game can automatically support the thousands of clothing items created by the community.
3. **Animation Reuse:** Even if you replace the model mesh, if your custom character uses the Citizen skeleton, you can reuse the complex Animgraph and hundreds of movement animations provided here.

## Visual Diagnostics
![Citizen Character](https://files.facepunch.com/maxlebled/1b1211b1/citizen_header.jpg)

When working with the Citizen, you can open `models/citizen/citizen.vmdl` in the ModelDoc editor to view all available animation sequences, bone names, and attachment points visually. 

## Sample Projects

The **Sweeper** sample (`game/samples/sweeper/`) demonstrates how to spawn a Citizen, apply the local player's custom avatar clothing, and hook up the `CitizenAnimationHelper` to drive its Animgraph using physics velocity.

## Troubleshooting

:::warning Common Gotchas
- **Model fails to load (pink/black checkerboard):**
  Ensure your project actually depends on `facepunch.citizen`. If the addon isn't mounted, the engine cannot find `models/citizen/citizen.vmdl`.
- **Animations aren't playing:**
  To play animations automatically, your GameObject must have a Component that sets parameters on the `SkinnedModelRenderer`. The `CitizenAnimationHelper` class (often found in the base addon) is typically used for this.
- **Clothing doesn't attach:**
  Clothing items must be parented to the GameObject with the `SkinnedModelRenderer`, and they must be configured as `Clothing` resources to merge properly with the Citizen skeleton.
:::

## Exploring the Source

If you want to understand how a AAA-quality character rig is built, explore the `game/addons/citizen/` directory:
- **`models/citizen/citizen.vmdl`:** Open this in the Model Editor to see the bone structure and physics hull setup.
- **`models/citizen/citizen.vanmgrph`:** Open this in the Animgraph Editor. This is the single most important file in the addon. It shows how the engine blends between walking, running, crouching, and aiming based on simple float parameters.

## Related Pages
- [Clothing System](../../../assets/clothing/index.md)

