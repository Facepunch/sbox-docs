---
title: "Attach an Object to a Bone"
icon: "🗡️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/SkinnedModelRenderer.cs
created: 2026-04-27
updated: 2026-04-27
---

# Attach an Object to a Bone

To attach an object like a weapon, hat, or accessory to a specific part of an animated character, you parent it to a "Bone Object" created by a `SkinnedModelRenderer`.

## Quick Working Example

```csharp
using Sandbox;

public sealed class EquipmentManager : Component
{
    [Property] public SkinnedModelRenderer CharacterRenderer { get; set; }
    [Property] public GameObject WeaponPrefab { get; set; }

    public void EquipWeapon()
    {
        if ( CharacterRenderer == null || WeaponPrefab == null ) return;

        // 1. Find the bone by name (e.g., "hold_R" for the right hand)
        GameObject rightHandBone = CharacterRenderer.GetBoneObject( "hold_R" );

        if ( rightHandBone.IsValid() )
        {
            // 2. Clone the weapon prefab
            GameObject newWeapon = WeaponPrefab.Clone();

            // 3. Parent it to the bone object
            newWeapon.SetParent( rightHandBone );

            // 4. Zero out its local transform so it perfectly aligns with the hand
            newWeapon.LocalPosition = Vector3.Zero;
            newWeapon.LocalRotation = Rotation.Identity;
        }
    }
}
```

### Unequipping an Item
To remove the weapon, destroy it. You do not need to destroy the bone object.

```csharp
public void DropWeapon( GameObject weapon )
{
    // Detach it from the hand
    weapon.SetParent( null );
    
    // Add physics so it falls
    weapon.Components.GetOrCreate<Rigidbody>();
    
    // Or just destroy it if it disappears
    // weapon.Destroy();
}
```

## Troubleshooting

:::warning "My weapon is floating weirdly away from the hand!"
This usually happens if you forgot to reset the local transform after parenting. Always set `LocalPosition = Vector3.Zero` and `LocalRotation = Rotation.Identity` to snap the weapon exactly to the bone's pivot point. If it is still offset, the weapon prefab's origin might be incorrect in its modeling software.
:::

:::danger "GetBoneObject returns an invalid object!"
Check the exact spelling of the bone name. Bone names are case-sensitive and must match exactly how they are defined in the imported `.vmdl` asset. Common names include `"hold_R"`, `"ValveBiped.Bip01_R_Hand"`, or `"head"`. You can inspect the model in the editor to find the correct names.
:::

## Related Pages
- [Spawn a Prefab](spawn-prefab.md)
- [Skinned Model Renderer](../scene/components/reference/skinnedmodelrenderer.md)
