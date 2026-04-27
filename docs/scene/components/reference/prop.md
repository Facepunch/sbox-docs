---
title: "Prop"
icon: "🧸"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/Prop.cs
updated: 2026-04-25
created: 2026-04-27
---

# Prop

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `Prop` component.

The `Prop` component is a high-level helper that automatically sets up rendering, physics, and damage handling for a 3D model.

## Quick Working Example

```csharp
using Sandbox;

public sealed class PropSpawner : Component
{
    [Property] public Model PropModel { get; set; }

    public void SpawnExplosiveBarrel( Vector3 position )
    {
        var go = new GameObject( true, "Explosive Barrel" );
        go.WorldPosition = position;

        var prop = go.Components.Create<Prop>();
        prop.Model = PropModel;
        
        // You can instantly blow it up via code
        prop.Kill();
    }
}
```

### Dealing Damage to a Prop

Props natively implement the `IDamageable` interface. If you hit them with a weapon or a physics collision, they will take damage.

```csharp
var prop = Components.Get<Prop>();

// Manually apply damage to the prop
var damageInfo = new DamageInfo( 50f, GameObject.WorldPosition, Vector3.Zero );
prop.OnDamage( damageInfo );

// Check if the prop is currently on fire
if ( prop.IsOnFire )
{
    Log.Info("The prop is burning!");
}
```

### Breaking and Exploding

You can programmatically trigger a prop's destruction events without dealing damage:

```csharp
var prop = Components.Get<Prop>();

// Immediately sets health to 0, breaks into gibs, and destroys the GameObject
prop.Kill();

// If the model is configured to explode, this triggers the explosion
prop.CreateExplosion();

// Sets the prop on fire (if its model data says it is flammable)
prop.Ignite();
```

## Configuration

| Property | Description |
|:---|:---|
| `Model` | The 3D model to use. The prop will read its physics and damage data. |
| `IsStatic` | If true, the prop will generate a static collider and will not move or simulate gravity. |
| `Health` | The current health of the prop. If it reaches 0, it breaks. |
| `StartAsleep` | If true, the physics body starts asleep, saving performance until something hits it. |
| `Tint` | An overall color tint applied to the model. |

## Troubleshooting

:::danger "I can't edit the Rigidbody on my Prop!"
Because the `Prop` component generates the `ModelRenderer`, `ModelCollider`, and `Rigidbody` dynamically (procedurally) based on the model file, you cannot edit them in the Inspector. If you need fine-grained control over physics or rendering, you should click the **Break into separate components** button on the Prop component, which deletes the `Prop` script and leaves the underlying procedural components fully editable.
:::

## Related Pages
- [Rigidbody](../../../systems/physics/rigidbody.md)
- [Deal and Take Damage](../../../how-to/deal-damage.md)
