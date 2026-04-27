---
title: Minimal Game Template
sources:
  - game/templates/game.minimal
updated: 2026-04-25
created: 2026-04-27
---

# Minimal Game Template

The **Minimal Game** template (`game/templates/game.minimal/`) is the recommended starting point for creating a new standalone, playable game in s&box.

## Quick Working Example

This template provides a basic starting point with a simple `MyComponent` script. Here is how you can use it to rotate your object, similar to what you might build when starting out:

```csharp
using Sandbox;

public sealed class MyComponent : Component
{
	[Property] public string StringProperty { get; set; }
	[Property] public float RotationSpeed { get; set; } = 90f;

	protected override void OnUpdate()
	{
        if ( !string.IsNullOrEmpty( StringProperty ) )
        {
            // Spin the object
		    Transform.LocalRotation *= Rotation.FromAxis( Vector3.Up, RotationSpeed * Time.Delta );
        }
	}
}
```

1. **Standalone Games:** Use this template when building a game meant to be played by end-users (e.g., an FPS, a puzzle game, a racing game).
2. **Custom Game Loops:** You will typically create a central `Component` attached to a root GameObject in your Scene to handle game states (like matching, playing, and round-over).
3. **Prefab Spawning:** The template demonstrates the pattern of defining a `[Property] GameObject Prefab` and cloning it into the world when needed.

## Walkthrough: Starting Your Game

To get your game running using this template:
1. Open the s&box Editor and click **New Project**.
2. Select **Minimal Game** and pick a folder.
3. Open the `Assets/scenes/minimal.scene` file from the Asset Browser.
4. The scene comes with a basic setup, often including a camera, lighting, and an example object.
5. In your `Code` folder, you'll find `MyComponent.cs`. You can attach this component to any GameObject in your scene.
6. Look at the Inspector on the right to see `StringProperty` exposed.
7. Press the green **Play** button at the top to run the scene.
8. In the `Editor` folder, you'll find `MyEditorMenu.cs`, demonstrating how to add custom tools to the s&box editor.

## Root Template Files

The project template infrastructure also provides several global templates located in the root of the templates folder (`game/templates/`). These are used as foundational building blocks when you create new files in the Editor:
- **`default.cs` & `component.cs`**: The boilerplate C# structures used when creating new scripts from the Editor context menu.
- **`default.razor`**: The starting markup for new UI panels, ensuring they inherit from `Panel` correctly.
- **Shaders (`compute.shader`, `material.shader`, `unlit.shader`)**: Basic templates for generating custom HLSL logic via ShaderGraph or code, ensuring they implement the required standard engine passes.

## Configuration

When using this template, you should configure your `.sbproj` settings:

| Setting | Purpose |
| :--- | :--- |
| **Type** | Must be set to `game`. |
| **Startup Scene** | The default `.scene` file the engine will load when the game is launched. |
| **Max Players** | Configure the maximum number of networked clients that can join a single server. |

## Performance

Since this is a full game template, managing performance becomes critical as the game grows. Be cautious of placing heavy logic in hot paths like `OnUpdate` or `OnFixedUpdate`. Use the SboxProfiler to track down frame-time spikes early in development. When spawning lots of prefabs (like bullets or enemies), consider implementing object pooling instead of repeatedly calling `Clone()`. 
If you find your main game thread stalling during player spawn (`Clone()`), verify that your PlayerPrefab does not contain excessively dense hierarchies or complex initializers that force synchronous Asset loading.

## Visual Diagnostics

Because this template provides an Editor folder (`game/templates/game.minimal/Editor/`), it demonstrates how to add custom menu items and tools to the s&box Editor. For instance, `MyEditorMenu.cs` adds an option to the editor menu. You can expand on this to create custom `[EditorTool]`s to visualize your specific game state (e.g., a custom window that graphs active player health or score) directly inside the s&box Editor, bypassing standard console logs.

- **Networking:** The template sets up a minimal game scene, but does not handle multiplayer synchronization out of the box. You must explicitly configure `GameObject.Network.SetOwner()` and sync properties via `[Sync]` attributes if you intend to add multiplayer support.
- **Save State Persistence:** Game state isn't saved automatically. Use `FileSystem.Data` or `Cookie` APIs to implement save features manually.

## Troubleshooting

:::warning Common Gotchas
- **"Type must be 'game' to be launched"**
  If you try to launch your project and get this error, check your Project Settings and ensure the Addon Type is set to Game, not Library or Addon.
- **Lighting is completely black:**
  The default scene comes with an Envmap Probe and a Directional Light. If you deleted them, add them back or add a `Skybox2D` component to your scene so the PBR renderer has light to reflect.
:::

## Sample Validation
Review `game/samples/sweeper/` for a concrete example of a project built using this template structure, expanding the single script into a full UI and Game Manager architecture.

## Related Pages
- [Scene System](../../scene/index.md)
- [Your First Project](../../getting-started/first-project.md)
