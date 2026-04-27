---
title: "Effects"
icon: "✨"
created: 2025-06-26
updated: 2026-04-25
---

# Effects

The Effects system allows you to create dynamic visual flourishes in your game, including particle systems, glowing beams, and impact decals.

## Quick Working Example

You can spawn effects from code by instantiating prefabs that have effect components (like `ParticleEffect` or `Decal`) attached.

```csharp
public class WeaponEffects : Component
{
	[Property] public GameObject BulletHolePrefab { get; set; }
	[Property] public GameObject MuzzleFlashPrefab { get; set; }

	public void FireWeapon( Vector3 hitPosition, Vector3 hitNormal )
	{
		// Spawn a muzzle flash particle effect at the gun's barrel
		MuzzleFlashPrefab.Clone( WorldPosition );

		// Spawn a bullet hole decal where the bullet hit
		var decal = BulletHolePrefab.Clone( hitPosition );
		
		// Align the decal with the surface normal
		decal.WorldRotation = Rotation.LookAt( hitNormal );
	}
}
```

## Navigation Hub

* [**Particle Effect**](particle-effect/index.md) - Learn about the heavily multithreaded CPU particle system for spawning sprites and meshes.
* [**Decals**](decals/index.md) - Project textures onto the environment for bullet holes, blood splatters, and graffiti.
* [**Beams**](beams/index.md) - Create animated line renderers for lasers, tracers, and lightning strikes.
* [**Indirect Light Volumes**](indirect-light-volumes.md) - Learn about baking bounce lighting and GI probes for your environments.
* [**Volumetric Fog Volume**](volumetric-fog-volume.md) - Add localized, physically-based volumetric fog and god rays.

## Related Topics

* [Temporary Effect](temporaryeffect.md) - Automatically clean up effect GameObjects when they finish playing.
