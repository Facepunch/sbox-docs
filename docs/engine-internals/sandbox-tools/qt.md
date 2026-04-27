---
title: "Qt"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Qt/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Tools.Qt

This section covers the internal implementation of the `Qt` subsystem within the s&box Editor.

## Architecture

This folder contains the massive C#-to-C++ interop layer that bridges s&box's managed environment with the native Qt framework. It is loaded only when the engine runs in tools mode (`sbox-dev.exe`).

Because s&box aims to provide a fully extensible Editor in C# without requiring C++ plugins, every major Qt UI concept (Windows, Layouts, Buttons, Events, Drag & Drop) is mapped into a C# equivalent class. 

## Core Concepts

### `QObject` and Memory Management

At the heart of the integration is the `Editor.QObject` base class, which wraps a native pointer (`Native.QObject`).
Memory management between C# (Garbage Collection) and C++ (Manual / Parent-Child hierarchy) is handled automatically:
- When a native Qt object is destroyed (e.g., a window is closed), it triggers a callback to `NativeShutdown()`, which unregisters the object, frees its `GCHandle`s, and marks the C# wrapper as invalid (`IsDestroyed = true`).
- If a C# script calls `.Destroy()`, it invokes `_object.deleteMuchLater()` on the native side to ensure safe deletion on the UI thread without crashing in the middle of an event loop.

### `Widget`

`Editor.Widget` inherits from `QObject` and is the base class for all visual elements.

When you instantiate a Widget in C#:
```csharp
var widget = new Widget( parentWidget );
```
1. It calls into the native interop to allocate a `QWidget`.
2. It assigns the `parentWidget` natively.
3. It stores the native pointer in `_widget`.

From there, layout additions and properties (like `.MinimumSize` or `.WindowName`) translate directly into native Qt calls.

### The Application and Event Loop

The `Editor.Application` class controls the top-level Qt application context.
- **Styling:** The Editor uses CSS-like stylesheets for Qt styling. `Application.ReloadStyles()` loads CSS files from the addons directory, parses custom Theme variables, and applies them to the native `QApp`.
- **Event Pumping:** s&box's main loop sometimes blocks the Qt thread. `Application.Spin()` is manually called by the engine during heavy operations to pump the Qt event loop, keeping the UI responsive and preventing the OS from marking the window as "Not Responding."

## Common Patterns

- **Callbacks & Delegates:** The engine creates native C++ delegates that invoke C# `Action`s via `GCHandle` and `Marshal.GetFunctionPointerForDelegate()`. These are heavily used for Qt's Signals & Slots mechanism (e.g., button clicks).
- **FindOrCreate:** When C++ passes a native pointer back to C#, the `QObject.FindOrCreate( Native.QObject )` method is used. It looks up the dictionary of all known objects (`AllObjects`). If the pointer belongs to a known C# wrapper, it returns it; otherwise, it constructs a new C# wrapper by inspecting the type.

## Troubleshooting

:::danger "Widget accessed from wrong thread"
Qt is strictly single-threaded. You cannot instantiate a `Widget` or modify its properties from an async `Task` or a background thread. Doing so will instantly crash the Editor. Always dispatch work back to the main thread.
:::

:::warning "Object was destroyed"
If you maintain a C# reference to a Widget that a user has closed, attempting to access its properties will throw an exception. Check `Widget.IsValid` before interacting with long-lived widget references.
:::
