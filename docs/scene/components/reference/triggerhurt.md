---
title: "Trigger Hurt"
icon: "🏥"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/TriggerHurt.cs
updated: 2026-04-25
created: 2026-04-27
---

# Trigger Hurt

> **New to physics?** Physics overview is at **[Physics](../../../systems/physics/index.md)**. This page is the reference for the `TriggerHurt` component.

The `TriggerHurt` component turns a collider into a damage zone that continuously hurts anything inside it over time, like a pool of lava or a toxic gas cloud.

## Quick Working Example

Usually you place a `TriggerHurt` in the Editor, but you can create one dynamically from code:

```csharp
public class SpawnLavaPool : Component
{
    protected override void OnStart()
    {
        var lavaObj = new GameObject( true, "LavaPool" );
        lavaObj.WorldPosition = WorldPosition;

        // 1. You must attach a Collider to define the area
        var box = lavaObj.Components.Create<BoxCollider>();
        box.Scale = new Vector3( 200, 200, 50 );
        box.IsTrigger = true; // Set to true so objects pass through it

        // 2. Add the TriggerHurt component
        var hurt = lavaObj.Components.Create<TriggerHurt>();
        hurt.Damage = 15f; // Deal 15 damage
        hurt.Rate = 0.5f;  // Every half second

        // Only hurt objects with the "player" or "enemy" tag
        hurt.Include.Add( "player" );
        hurt.Include.Add( "enemy" );
    }
}
```

## Configuration

| Property | Description |
|:---|:---|
| `Damage` | How much damage is applied each tick. |
| `Rate` | The delay (in seconds) between each instance of damage. |
| `DamageTags` | A set of tags added to the `DamageInfo` object (e.g., `fire`, `acid`, `fall`) to help the receiving object decide how to react. |
| `Include` | If this is not empty, only objects that possess at least one of these tags will be hurt. |
| `Exclude` | Objects possessing any of these tags will be completely ignored and will not receive damage. |

## Troubleshooting

:::danger "It's not doing any damage!"
The `TriggerHurt` component **requires** a `Collider` component (like `BoxCollider` or `SphereCollider`) on the exact same GameObject to function. If you don't have a collider, the trigger doesn't know what its volume is. Additionally, ensure the objects entering it implement the `IDamageable` interface.
:::

:::warning "Damage happening twice in multiplayer"
If you wrote your own `IDamageable` implementation and you are seeing damage applied multiple times, check if you are accidentally running damage logic on the client. `TriggerHurt` automatically only runs on the `Networking.IsHost`, but your receiver must also respect authority.
:::

## Related Pages
- [Hitboxes](hitboxes.md)
- [Deal and Take Damage](../../../how-to/deal-damage.md)
