---
title: "Logging & Debugging"
icon: "🐛"
created: 2026-05-05
updated: 2026-05-05
---

# Logging & Debugging

## Log

`Log` writes messages to the in-game console and output window. There are four levels:

```csharp
Log.Info( "Player spawned" );
Log.Warning( "Health is zero, expected > 0" );
Log.Error( "Failed to find spawn point" );
Log.Trace( "OnUpdate tick" );  // verbose, filtered out by default
```

You can log any object - if it isn't a primitive it becomes an inspectable link in the console.

```csharp
Log.Info( GameObject );
Log.Info( $"Position: {WorldPosition}, Speed: {velocity.Length}" );
```

Logging exceptions is supported directly:

```csharp
try { DoSomething(); }
catch ( Exception e ) { Log.Error( e ); }
```

Clicking on a log message in the editor console will show a stack trace and allow you to jump to the source code.

## DebugOverlay

`DebugOverlay` draws wireframe shapes and text in the game world at runtime. It's accessed via `DebugOverlay` from any component.

Draws are single-frame by default. Pass a `duration` to keep them visible longer.

```csharp
// Line between two points
DebugOverlay.Line( from, to, Color.White );

// Sphere at a position
DebugOverlay.Sphere( new Sphere( WorldPosition, 50f ), Color.Green );

// Box
DebugOverlay.Box( WorldPosition, new Vector3( 32, 32, 32 ), Color.Red );

// Text in the world at a 3D position
DebugOverlay.Text( WorldPosition, "Hello", 24 );

// Text pinned to the screen (pixel coordinates)
DebugOverlay.ScreenText( new Vector2( 50, 50 ), $"Speed: {velocity.Length:F1}" );
```

There are a variety of shapes and visualizations available - see the API reference for details.

Use `duration` to keep a draw visible for more than one frame:

```csharp
// Stay visible for 2 seconds
DebugOverlay.Sphere( new Sphere( hitPos, 8f ), Color.Yellow, duration: 2f );
```

## Gizmos

Gizmos draw shapes in the **editor only** - they are never visible in a running game. Override `DrawGizmos()` on your component to add them.

```csharp
protected override void DrawGizmos()
{
    Gizmo.Draw.Color = Color.Green;
    Gizmo.Draw.LineSphere( Vector3.Zero, 50f );

    Gizmo.Draw.Color = Color.Red;
    Gizmo.Draw.LineBBox( new BBox( -16, 16 ) );

    Gizmo.Draw.Line( Vector3.Zero, Vector3.Up * 100f );
}
```

If you really want to draw gizmos in a running game, you can use `Gizmo.Draw` in `OnUpdate()` - they only draw for a single frame.

Draws are in **local space** relative to the component's transform by default. Gizmo settings are scoped to the current `DrawGizmos()` call, so you can change colors and other properties between draws.

Change to world space with `Gizmo.Transform`:

```csharp
Gizmo.Transform = global::Transform.Zero;
```

There are a lot of other gizmo visualizations available, including editor handles for moving and rotating objects. See the API reference for details, as there are too many to cover here.

:::tip
Use Gizmos to visualise component properties like range, radius, or detection zones while building your game - they won't affect performance in a build.
:::
