---
title: "Scene Information"
icon: "ℹ️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Utility/SceneInformation.cs
updated: 2026-04-25
created: 2026-04-27
---

# Scene Information

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `SceneInformation` component.

The `SceneInformation` component allows you to embed metadata (like Title, Author, and Version) into a Scene, which can then be read without needing to fully load the scene.

## Quick Working Example

```csharp
using Sandbox;

public class MetadataReader : Component
{
    protected override void OnStart()
    {
        // Find all scene files
        var allScenes = ResourceLibrary.GetAll<SceneFile>();

        foreach ( var scene in allScenes )
        {
            // We can read the metadata that was set by the SceneInformation component
            string title = scene.GetMetadata( "Title", "Unknown Title" );
            string author = scene.GetMetadata( "Author", "Unknown Author" );

            Log.Info( $"Found scene: {title} by {author}" );
        }
    }
}
```

## Configuration

| Property | Description |
|:---|:---|
| `Title` | The display name of the scene. |
| `SceneTags` | Tags to categorize the scene (e.g., "multiplayer", "sandbox"). |
| `Group` | A grouping identifier, useful for organizing scenes in UI menus. |
| `Version` | The version string of the scene. |
| `Author` | The creator of the scene. |
| `Description` | A detailed text description. |
| `Changes` | Patch notes or recent changes made to the scene. |

## Related Pages
- [Scene Metadata](../../scenes/scene-metadata.md)
