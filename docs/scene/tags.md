---
title: "Tags"
icon: "🏷️"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameTags/GameTags.cs
  - engine/Sandbox.Engine/Utility/ITagSet.cs
created: 2026-04-27
updated: 2026-04-27
---

# Tags

GameTags are simple strings you can attach to GameObjects to easily categorize, filter, and identify them at runtime.

## Quick Working Example

```csharp
public class Spawner : Component
{
    protected override void OnStart()
    {
        var enemy = new GameObject();
        
        // Add a tag to categorize this object
        enemy.Tags.Add( "enemy" );
        
        // Add multiple tags at once
        enemy.Tags.Add( "boss", "fire_type" );

        // Later, check if an object has a tag
        if ( enemy.Tags.Has( "boss" ) )
        {
            Log.Info("A boss has spawned!");
        }
    }
}
```

### Filtering Physics Traces

Tags are heavily used in the Physics system to determine what objects a trace (raycast) should hit or ignore.

```csharp
// Shoot a ray that ONLY hits objects tagged "solid" or "glass"
var tr = Scene.Trace.Ray( startPos, endPos )
    .WithAnyTags( "solid", "glass" )
    .Run();

if ( tr.Hit )
{
    Log.Info( $"Hit: {tr.GameObject.Name}" );
}
```

### Removing and Toggling Tags

You can dynamically change tags at runtime to reflect state changes.

```csharp
// Remove a specific tag
GameObject.Tags.Remove( "invincible" );

// Clear all tags from this GameObject
GameObject.Tags.RemoveAll();

// Toggle a tag on or off
GameObject.Tags.Set( "poisoned", true );
```

## Troubleshooting

:::warning "Tags are case-insensitive"
Under the hood, all tags are converted to lowercase. Checking for `Tags.Has("Player")` is exactly the same as checking for `Tags.Has("player")`. It is recommended to use lowercase strings to avoid confusion.
:::

## Related Pages
* [GameObject](gameobject.md)
* [Physics Traces](../systems/physics/collisions-and-triggers.md)
