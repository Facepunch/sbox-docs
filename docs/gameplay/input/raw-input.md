---
title: "Raw Input"
icon: "⌨️"
created: 2025-04-07
updated: 2026-09-03
---

# Raw Input

:::tip
We very much recommend that you only use this if you can't use input actions for whatever reason. You can't rebind these at all.

:::


We have the ability to access keyboard inputs directly, bypassing input actions. 

```csharp
//
// Is the W key down this frame?
//
if ( Input.Keyboard.Down( "W" ) )
{
    Log.Info( "W is down!" );
}

//
// Was the W key pressed this frame?
//
if ( Input.Keyboard.Pressed( "W" ) )
{
    Log.Info( "W was pressed!" );
}

//
// Was the W key released this frame?
//
if ( Input.Keyboard.Released( "W" ) )
{
    Log.Info( "W was released!" );
}
```

Most of the keys on the keyboard should work, here's an exhaustive list of them.

| Key | Name |
|-----|------|
| "0" - "9" | 0 - 9 |
| "a" - "z" | A - Z |
| "KP_0" - "KP_9" | Numpad 0 - Numpad 9 |
| "KP_DIVIDE" | Numpad / |
| "KP_MULTIPLY" | Numpad \* |
| "KP_MINUS" | Numpad - |
| "KP_PLUS" | Numpad + |
| "KP_ENTER" | Numpad Enter |
| "KP_DEL" | Numpad Delete |
| "<" | Less Than |
| "\[" | Left Bracket |
| "\]" | Right Bracket |
| "SEMICOLON" | Semicolon |
| " ' " | Apostrophe |
| " \` " | Backtick / Console Key |
| "," | Comma |
| "." | Period |
| "/" | Slash |
| "\\\\" | Backslash |
| "-" | Hyphen / Minus |
| "=" | Equals |
| "ENTER" | Enter |
| "SPACE" | Space |
| "BACKSPACE" | Backspace |
| "TAB" | Tab  |
| "CAPSLOCK" | Caps Lock |
| "NUMLOCK" | Num Lock |
| "ESCAPE" | Escape |
| "SCROLLLOCK" | Scroll Lock |
| "INS" | Insert |
| "DEL" | Delete |
| "HOME" | Home |
| "END" | End  |
| "PGUP" | Page Up |
| "PGDN" | Page Down |
| "PAUSE" | Pause |
| "SHIFT" | Left Shift |
| "RSHIFT" | Right Shift |
| "ALT" | Left Alt |
| "RALT" | Right Alt |
| "UPARROW" | Up Arrow |
| "LEFTARROW" | Left Arrow |
| "RIGHTARROW" | Right Arrow |
| "DOWNARROW" | Down Arrow |
