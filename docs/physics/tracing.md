---
title: "Tracing"
icon: "🎯"
created: 2023-12-28
updated: 2024-05-15
---

# Tracing

Scenes can be traced against using `Scene.Trace` - which uses a builder pattern to make construction a bit easier. At a minimum, traces have a shape, start, and end. You can also filter which specific tags will be hit or ignored, or opt-in to using your project's collision rule matrix by calling `WithCollisionRules(tag)`.

Here are some examples.

# Simplest Trace

```csharp
SceneTraceResult tr = Scene.Trace.Ray( startPos, endPos ).Run();

if ( tr.Hit )
{
	Log.Info( $"Hit: {tr.GameObject} at {tr.EndPosition}" );
}
```

# Use Collision Rules

This will fire a ray using the collision rules of a `bullet` tag, as configured in your project's Collision settings.

```csharp
var tr = Scene.Trace
	.Ray( startPos, endPos )
 	.WithCollisionRules( "bullet" ) // Hits everything that a bullet would hit
 	.Run();
```

# Sphere Trace

```csharp
var tr = Scene.Trace
	.Sphere( 32.0f, startPos, endPos ) // 32 is the radius
	.WithoutTags( "player" ) // ignore GameObjects with this tag
	.Run();
```

# Box Trace

```csharp
var tr = Scene.Trace
	.Ray( start, end )
	.Size( new BBox( -5, 5 ) ) // size of the aabb
	.UseHitboxes( true ) // hit hitboxes too!
	.Run();
```

# Debugging traces

You can visualize traces in the editor by calling `DebugOverlay.Trace(tr)` - this will draw the trace and the hit point if there was a hit.

```csharp
var tr = Scene.Trace
    .Ray( start, end )
    .Run();

DebugOverlay.Trace( tr, 5f ); // lasts for 5 seconds
```
