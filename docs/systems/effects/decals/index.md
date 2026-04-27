---
title: "Decals"
icon: "🔣"
created: 2025-06-26
updated: 2026-04-25
---

# Decals

Decals are a fast, performant way to project a texture onto surrounding geometry.

## Quick Working Example

```csharp
public class FireWeapon : Component
{
    [Property] public GameObject BulletHolePrefab { get; set; }

    public void OnFire( SceneTraceResult hit )
    {
        // Clone a prefab containing a DecalComponent
        var decalObject = BulletHolePrefab.Clone( hit.HitPosition );
        
        // Orient the decal to face away from the surface it hit
        decalObject.WorldRotation = Rotation.LookAt( hit.Normal );
    }
}
```

## Navigation Hub

* [**Decal Component**](decal-component.md) - The core `Decal` component that draws the decal in the game world.
* [**DecalDefinition**](decaldefinition.md) - The asset file that dictates the texture, material properties, and variance of a decal.
* [**Animated Effects**](animated-effects.md) - How to animate a decal's properties over time.
* [**Lifetime**](lifetime.md) - How to manage decal fading and transient cleanups for performance.
