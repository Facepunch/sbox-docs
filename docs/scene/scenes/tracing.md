---
title: "Tracing"
icon: "🎯"
created: 2023-12-28
updated: 2026-04-11
sources:
  - engine/Sandbox.Engine/Scene/Scene/Scene.Trace.cs
---

# Tracing

Tracing (or Raycasting) is how you fire invisible lines or shapes through the world to see what they hit.

```csharp
// Fire a simple line straight down from this object
var tr = Scene.Trace.Ray( GameObject.WorldPosition, GameObject.WorldPosition + Vector3.Down * 1000f )
    .Run();

if ( tr.Hit )
{
    Log.Info( $"Hit: {tr.GameObject.Name} at {tr.HitPosition}" );
}
```

### Multiple Hits

By default, `Run()` returns the first thing the trace hits. If you want to penetrate objects and get a list of everything the trace passes through, use `RunAll()`, which returns an `IEnumerable<SceneTraceResult>`.

```csharp
IEnumerable<SceneTraceResult> results = Scene.Trace.Ray( startPos, endPos ).RunAll();

foreach ( var tr in results )
{
    Log.Info( $"Hit: {tr.GameObject}" );
}
```

### Sphere Trace

You can cast a sphere instead of a point line, which is useful for things like thick laser beams or checking if a larger object can fit through a space.

```csharp
var tr = Scene.Trace
    .Sphere( 32.0f, startPos, endPos ) // 32 is the radius
    .WithoutTags( "player" ) // ignore GameObjects with this tag
    .Run();
```

### Box Trace

Similar to a sphere, you can trace a bounding box through the world.

```csharp
var tr = Scene.Trace
    .Box( new Vector3( 10, 10, 10 ), startPos, endPos ) // extents
    .Run();
```

## Configuration

You can filter what the trace hits using tags or your project's collision rules.

```csharp
var tr = Scene.Trace
    .Ray( startPos, endPos )
    .WithCollisionRules( "bullet" ) // Hits everything that a bullet would hit
    .Run();
```

| Method | Description |
|---|---|
| `WithoutTags( params string[] tags )` | Ignores GameObjects with any of the specified tags. |
| `WithAnyTags( params string[] tags )` | Only hits GameObjects that have at least one of the specified tags. |
| `WithCollisionRules( string tag )` | Uses the collision matrix defined in your project settings for the given tag. |
| `IgnoreGameObjectHierarchy( GameObject obj )` | Ignores a specific GameObject and all its children. |

### SceneTraceResult Properties

When you call `Run()`, you receive a `SceneTraceResult` struct containing details about the hit.

| Property | Description |
|---|---|
| `Hit` | `true` if the trace hit something, `false` if it reached the end point without hitting anything. |
| `HitPosition` | The exact `Vector3` point in the world where the hit occurred. |
| `Normal` | The surface normal at the hit position. |
| `StartedSolid` | `true` if the trace started inside a solid collider. |
| `GameObject` | The `GameObject` that was hit. |
| `Component` | The specific `Component` (usually a `Collider`) that was hit. |
| `Distance` | The distance from the start point to the hit point. |

## Troubleshooting

:::warning
**Trace hitting the object firing it?**
If your trace originates from a player or weapon, it will likely hit that object first. Use `IgnoreGameObjectHierarchy( GameObject )` or `WithoutTags( "player" )` to filter it out.
:::

## Related Pages
* [GameObject](../gameobject.md)
* [Colliders](../../systems/physics/colliders.md)
