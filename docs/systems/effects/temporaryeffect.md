---
title: "TemporaryEffect"
icon: "💨"
sources:
  - engine/Sandbox.Engine/Scene/Components/Effects/TemporaryEffect.cs
created: 2026-04-27
updated: 2026-04-27
---

# TemporaryEffect

Automatically destroys a GameObject after a set time, optionally waiting for sounds and particles to finish first.

## Quick Working Example

```csharp
public sealed class ExplosionSpawner : Component
{
	[Property] public GameObject ExplosionPrefab { get; set; }

	public void SpawnExplosion( Vector3 position )
	{
		var explosion = ExplosionPrefab.Clone( position );
		
		// Ensure it cleans itself up after 2 seconds, 
		// but wait if particles are still emitting.
		var tempEffect = explosion.Components.GetOrCreate<TemporaryEffect>();
		tempEffect.DestroyAfterSeconds = 2.0f;
		tempEffect.WaitForChildEffects = true;
	}
}
```

### Fire and Forget Prefabs
The most common way to use `TemporaryEffect` is simply attaching it to a Prefab in the Editor. For example, if you have a "Spark" particle prefab, add `TemporaryEffect` to the root, check "Wait For Child Effects", and set "Destroy After Seconds" to a low value like `0.5`. When you spawn this prefab in code, it will automatically clean itself up without you writing any extra code.

### Orphaned Effects
If you have an effect attached to a character (like a burning fire particle), and the character is destroyed, normally the particle would be destroyed instantly with it. By checking `BecomeOrphan`, the `TemporaryEffect` will detach itself from the character right before the character dies, stop looping, and let the remaining particles finish gracefully before finally destroying itself.

## Configuration

| Property | Description |
|:---|:---|
| `DestroyAfterSeconds` | The number of seconds to wait before attempting to destroy the GameObject. |
| `WaitForChildEffects` | If true, it checks if any component on this object or its children implements `ITemporaryEffect`. It will wait for `IsActive` to become false before destroying. |
| `BecomeOrphan` | If the parent GameObject is destroyed, this object will unparent itself, stop any looping effects, and wait to die gracefully. |

## Troubleshooting

:::danger Unending Loops
If you check `WaitForChildEffects` and you have a `ParticleEmitter` set to loop indefinitely, the `TemporaryEffect` will never destroy the object! Ensure your particles or sounds have a fixed duration or use `BecomeOrphan` to force them to stop looping.
:::

## Related Pages
* [Component Interfaces](../../scene/components/component-interfaces/index.md)
