---
title: "Detect Player Input"
icon: "🎮"
sources:
  - engine/Sandbox.Engine/Systems/Input/Input.cs
updated: 2026-04-25
created: 2026-04-27
---

# Detect Player Input

To respond to keyboard, mouse, or controller buttons in your game, use the static `Input` class.

## Quick Working Example

```csharp
using Sandbox;

public sealed class PlayerMovement : Component
{
	[Property] public float Speed { get; set; } = 300f;

	protected override void OnUpdate()
	{
		// 1. Check if an action button is currently held down
		if ( Input.Down( "jump" ) )
		{
			// Add upward force or trigger a jump animation
			Log.Info( "Jump button is held!" );
		}

		// 2. Check if a button was pressed exactly on this frame
		if ( Input.Pressed( "attack" ) )
		{
			// Fire a weapon
			Log.Info( "Attack button was pressed!" );
		}

		// 3. Read analog movement (WASD or left thumbstick)
		Vector3 movementInput = Input.AnalogMove;
		
		if ( !movementInput.IsNearlyZero() )
		{
			// Move the GameObject based on input
			GameObject.WorldPosition += movementInput * Speed * Time.Delta;
		}
	}
}
```

### Getting Look Direction (Mouse or Right Thumbstick)

To rotate a camera or player character, use `Input.AnalogLook`. It returns an `Angles` struct representing the relative rotation the player performed this frame.

```csharp
protected override void OnUpdate()
{
	Angles lookAngles = Input.AnalogLook;
	
	// Apply the look input to our current rotation
	GameObject.WorldRotation *= Rotation.From( lookAngles );
}
```

### Overriding the Pause Menu

By default, pressing the "Escape" key opens the engine's built-in pause menu. You can catch this input and handle it yourself (like opening your own custom inventory or menu).

```csharp
protected override void OnUpdate()
{
	if ( Input.EscapePressed )
	{
		// Tell the engine we handled the escape key
		Input.EscapePressed = false;
		
		// Run our own logic
		ToggleCustomMenu();
	}
}
```

## Troubleshooting

:::warning My input check is never true!
If `Input.Down("fire")` is never triggering, ensure the action name matches exactly what is configured in your project settings. Action names are strings and must be mapped to at least one key or button in the Editor.
:::

:::danger Input.Pressed triggering logic multiple times?
`Input.Pressed()` returns `true` for the *entire duration of the frame* in which the user presses the button. If you check it inside a loop or an event that fires multiple times per frame, your logic will execute multiple times because `Input.Pressed()` will continually return `true` for that whole frame. Stick to checking input once per frame inside `OnUpdate`.
:::

## Related Pages
- [Input System (Concepts)](../systems/input/index.md)
- [Basic Player Movement](player-movement.md) — apply input to a `CharacterController`.
- [Camera Follow Player](camera-follow.md)
