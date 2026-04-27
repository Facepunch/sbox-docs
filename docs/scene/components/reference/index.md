---
title: "Built-in Components"
icon: "🛠️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
updated: 2026-04-25
created: 2026-04-27
---

# Built-in Components

Each page describes one component's properties, lifecycle, and API. Look here when you know what you want to use and need its surface; for *how* to use components in general, see [Components](../index.md); for *learning* by building, see [Tutorials](../../../tutorials/index.md).

Every component s&box ships with. Each links to its own reference page with properties, lifecycle notes, common patterns, and gotchas.

## Rendering

- [**Model Renderer**](modelrenderer.md) — render a static 3D model.
- [**Skinned Model Renderer**](skinnedmodelrenderer.md) — render a skeletal animated model.
- [**Sprite Renderer**](spriterenderer.md) — animated 2D sprite in 3D space.
- [**Line Renderer**](linerenderer.md) — connect a list of points with a 3D line.
- [**Trail Renderer**](trailrenderer.md) — ribbon trail behind a moving object.
- [**Cable Component**](cablecomponent.md) — procedural cable between two anchors.
- [**Cable & Rope**](cable-rope.md) — combined verlet-rope + cable-node reference.
- [**Particle Renderers**](particle-renderers.md) — model / sprite / text / trail particle subtypes.
- [**Highlight Outline**](highlightoutline.md) — selection-style outline overlay.
- [**Decal**](decal.md) — project a texture onto surfaces.
- [**Text Renderer**](textrenderer.md) — render 3D text in the world.

## Lighting & Atmosphere

- [**Lights**](lights.md) — `PointLight`, `SpotLight`, `DirectionalLight` references.
- [**Ambient Light**](ambientlight.md) — non-directional ambient light source.
- [**Indirect Light Volume**](indirectlightvolume.md) — baked indirect lighting volume.
- [**Envmap Probe**](envmapprobe.md) — captures environment for reflections.
- [**Cubemap Fog**](cubemapfog.md) — image-based fog applied via skybox material.
- [**Gradient Fog**](gradientfog.md) — color-gradient fog.
- [**2D Skybox**](skybox2d.md) — 2D skybox + global indirect lighting.

## Physics

- [**Character Controller**](charactercontroller.md) — kinematic player movement (no Rigidbody).
- [**Player Controller**](player-controller.md) — full first/third-person controller built on `CharacterController`.
- [**Model Physics**](modelphysics.md) — multi-body physics from a model's joints (ragdolls).
- [**Spring Joint**](springjoint.md) — spring constraint between two bodies.
- [**Physics Filter**](physicsfilter.md) — collision-filter component.
- [**Hitboxes**](hitboxes.md) — overview of damage-zone components.
- [**Manual Hitbox**](manualhitbox.md) — hand-placed hitbox volumes.
- [**Model Hitboxes**](modelhitboxes.md) — hitboxes derived from a skeletal model.
- [**Trigger Hurt**](triggerhurt.md) — trigger volume that damages objects over time.
- [**Radius Damage**](radiusdamage.md) — explosion-style radial damage.

## Audio

- [**Sound Point**](soundpointcomponent.md) — plays a sound at a point in the world.
- [**Soundscape Trigger**](soundscapetrigger.md) — switches the active soundscape on enter.
- [**Audio Listener**](audiolistener.md) — overrides where the local client hears audio from.
- [**Lip Sync**](lipsync.md) — drives morph weights from sound spectrum.
- [**Voice**](voice.md) — voice-chat input/output.

## Maps & Geometry

- [**Map Instance**](mapinstance.md) — loads a map into the scene.
- [**Mesh Component**](meshcomponent.md) — runtime mesh component (created by the Mapping tool).
- [**Polygon Mesh**](polygonmesh.md) — editable half-edge mesh data type.
- [**Hammer Mesh**](hammer-mesh.md) — legacy Hammer-driven mesh component.
- [**Terrain**](terraincomponent.md) — heightmap-based terrain.
- [**Clutter Component**](cluttercomponent.md) — procedural clutter (grass, rocks).

## Camera & Scene

- [**Camera Component**](cameracomponent.md) — viewpoint for rendering.
- [**Scene Information**](sceneinformation.md) — metadata readable without loading the scene.
- [**Spawn Point**](spawnpoint.md) — placeable spawn marker.

## Navigation

- [**NavMesh Agent**](navmeshagent.md) — AI agent that pathfinds on the NavMesh.

## Effects

- [**Particle Effect**](../../../systems/effects/particle-effect/index.md) — CPU-simulated particles; pair with an emitter and a renderer.
- [**Beam Effect**](../../../systems/effects/beams/beameffect.md) — animated line for lasers, tracers, lightning.
- [**Temporary Effect**](../../../systems/effects/temporaryeffect.md) — destroys a GameObject after a delay (waits for child effects to finish).

## Game-genre built-ins

- [**Prop**](prop.md) — auto-configures rendering, physics, and damage for a model.
- [**Chair**](chair.md) — sittable chair built-in.
- [**Dresser**](dresser.md) — interactive dresser built-in.
- [**Fire Damage**](firedamage.md) — area fire damage.

## Special

- [**Missing Component**](missingcomponent.md) — placeholder that preserves data when a component type fails to load.

## Adding components from code

```csharp
// Always make a new one
var rb = GameObject.Components.Create<Rigidbody>();

// Get it if it exists, null otherwise
var rd = GameObject.Components.Get<ModelRenderer>();

// Get it or add it
var col = GameObject.Components.GetOrCreate<BoxCollider>();

// All of a type, walking the descendant tree
var allLights = GameObject.Components.GetAll<PointLight>(
    FindMode.EverythingInSelfAndDescendants );
```

When your component requires another component to function, use `[RequireComponent]`:

```csharp
public class EnemyAI : Component
{
    [RequireComponent] public CharacterController Movement { get; set; }

    protected override void OnUpdate()
    {
        Movement.MoveTo( WorldPosition + Vector3.Forward * 100f );
    }
}
```

The engine guarantees `Movement` is non-null by `OnAwake`. If the GameObject doesn't have a `CharacterController`, one is added automatically.

## Common gotcha

`Components.Get<T>()` returns `null` if the component doesn't exist. Always null-check or use `GetOrCreate<T>()`.

## Related Pages

- [Components Overview](../index.md) — what components are, how lookup works
- [Component Lifecycle](../component-lifecycle.md) — when each method fires
- [Component Methods Reference](../component-methods.md) — every overridable method
- [GameObject](../../gameobject.md) — the container components attach to
