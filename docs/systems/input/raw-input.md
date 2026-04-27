---
title: "Raw Input"
icon: "⌨️"
created: 2025-04-07
updated: 2026-04-11
sources:
  - engine/Sandbox.Engine/Systems/Input/Input.Keyboard.cs
---

# Raw Input

The `Input.Keyboard` API allows you to access specific hardware keys directly, bypassing the flexible action-based input system.

## Quick Working Example

```csharp
public sealed class MyCheatComponent : Component
{
    protected override void OnUpdate()
    {
        // Check if a specific physical key is held down
        if ( Input.Keyboard.Down( "W" ) )
        {
            Log.Info( "W is down!" );
        }
    }
}
```

### Querying Raw Keys

You can check if a specific key is pressed, held, or released using its string identifier.

```csharp
// Was the W key pressed exactly this frame?
if ( Input.Keyboard.Pressed( "W" ) )
{
    Log.Info( "W was just pressed!" );
}

// Is the W key currently being held down?
if ( Input.Keyboard.Down( "W" ) )
{
    Log.Info( "W is currently held down!" );
}

// Was the W key released exactly this frame?
if ( Input.Keyboard.Released( "W" ) )
{
    Log.Info( "W was just released!" );
}
```

## Configuration

Most of the keys on the keyboard should work. Here is an exhaustive list of the raw key names you can use.

| Key | Name | Key | Name |
|-----|------|-----|------|
| "0" - "9" | 0 - 9 | "TAB" | Tab  |
| "a" - "z" | A - Z | "CAPSLOCK" | Caps Lock |
| "KP_0" - "KP_9" | Numpad 0 - Numpad 9 | "NUMLOCK" | Num Lock |
| "KP_DIVIDE" | Numpad / | "ESCAPE" | Escape |
| "KP_MULTIPLY" | Numpad \* | "SCROLLLOCK" | Scroll Lock |
| "KP_MINUS" | Numpad - | "INS" | Insert |
| "KP_PLUS" | Numpad + | "DEL" | Delete |
| "KP_ENTER" | Numpad Enter | "HOME" | Home |
| "KP_DEL" | Numpad Delete | "END" | End  |
| "<" | Less Than | "PGUP" | Page Up |
| ">" | More Than | "PGDN" | Page Down |
| "\[" | Left Bracket | "PAUSE" | Pause |
| "\]" | Right Bracket | "SHIFT" | Left Shift |
| "SEMICOLON" | Semicolon | "RSHIFT" | Right Shift |
| " ' " | Apostrophe | "ALT" | Left Alt |
| " \` " | Backtick / Console Key | "RALT" | Right Alt |
| "," | Comma | "UPARROW" | Up Arrow |
| "." | Period | "LEFTARROW" | Left Arrow |
| "/" | Slash | "RIGHTARROW" | Right Arrow |
| "\\\\" | Backslash | "DOWNARROW" | Down Arrow |
| "-" | Hyphen / Minus | | |
| "=" | Equals | | |
| "ENTER" | Enter | | |
| "SPACE" | Space | | |
| "BACKSPACE" | Backspace | | |

## Troubleshooting

:::tip Best Practice
We very much recommend that you only use Raw Input if you absolutely cannot use input actions. Hardcoding keys means players cannot rebind them, which hurts accessibility and is frustrating for international keyboard layouts.
:::

:::warning Invalid Key Names
If a key is never registering as pressed, ensure you are using the exact string from the table above (e.g., using "Space" instead of "SPACE" might cause it not to be found).
:::

## Related Pages
* [Input (Actions)](index.md)
